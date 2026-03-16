---
title: "External Recon Automation: Scripts That Save Hours"
date: 2026-01-05
categories: Tools Red-team
excerpt: "Automated recon workflow - subdomain enumeration, port scanning, screenshots, tech fingerprinting, and cloud asset discovery."
tags:
  - reconnaissance
  - automation
  - bash
  - osint
  - bug-bounty
toc: true
toc_sticky: true
header:
  teaser: /assets/images/tools-scripts.svg
  overlay_image: /assets/images/tools-scripts.svg
  overlay_filter: 0.7
---

Reconnaissance is where engagements are won or lost. Miss a subdomain, overlook an exposed service, skip a forgotten dev environment - and you miss your way in. Manual recon doesn't scale. These scripts automate the boring parts so you can focus on what matters.

## The Recon Wrapper

This is my main script. It chains subdomain enumeration, port scanning, screenshot capture, and content discovery into one workflow.

```bash
#!/bin/bash
#
# recon.sh - Automated external reconnaissance
# Usage: ./recon.sh target.com
#

set -e

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
CYAN='\033[0;36m'
NC='\033[0m'

TARGET=$1
DATE=$(date +%Y%m%d)
OUTDIR="./recon_${TARGET}_${DATE}"
THREADS=50

banner() {
    echo -e "${CYAN}"
    echo "================================================"
    echo "  RECON AUTOMATION"
    echo "  Target: $TARGET"
    echo "  Output: $OUTDIR"
    echo "================================================"
    echo -e "${NC}"
}

check_tools() {
    local tools=("subfinder" "amass" "httpx" "nmap" "gowitness" "feroxbuster" "nuclei")
    for tool in "${tools[@]}"; do
        if ! command -v $tool &> /dev/null; then
            echo -e "${RED}[-] Missing: $tool${NC}"
            exit 1
        fi
    done
    echo -e "${GREEN}[+] All tools present${NC}"
}

subdomain_enum() {
    echo -e "\n${YELLOW}[*] Phase 1: Subdomain Enumeration${NC}"

    mkdir -p "$OUTDIR/subdomains"

    # Passive sources with subfinder
    echo "[*] Running subfinder..."
    subfinder -d $TARGET -silent -o "$OUTDIR/subdomains/subfinder.txt"

    # Amass passive
    echo "[*] Running amass passive..."
    timeout 600 amass enum -passive -d $TARGET -o "$OUTDIR/subdomains/amass.txt" 2>/dev/null || true

    # Combine and dedupe
    cat "$OUTDIR/subdomains/"*.txt 2>/dev/null | sort -u > "$OUTDIR/subdomains/all_subs.txt"

    local count=$(wc -l < "$OUTDIR/subdomains/all_subs.txt")
    echo -e "${GREEN}[+] Found $count unique subdomains${NC}"
}

resolve_alive() {
    echo -e "\n${YELLOW}[*] Phase 2: Resolving & Finding Live Hosts${NC}"

    mkdir -p "$OUTDIR/alive"

    # Resolve and probe HTTP/HTTPS
    echo "[*] Probing for live hosts..."
    cat "$OUTDIR/subdomains/all_subs.txt" | \
        httpx -silent -threads $THREADS \
        -status-code -content-length -title \
        -o "$OUTDIR/alive/httpx_full.txt"

    # Extract just URLs
    cat "$OUTDIR/alive/httpx_full.txt" | awk '{print $1}' > "$OUTDIR/alive/live_urls.txt"

    local count=$(wc -l < "$OUTDIR/alive/live_urls.txt")
    echo -e "${GREEN}[+] Found $count live URLs${NC}"
}

port_scan() {
    echo -e "\n${YELLOW}[*] Phase 3: Port Scanning${NC}"

    mkdir -p "$OUTDIR/ports"

    # Extract unique IPs
    cat "$OUTDIR/subdomains/all_subs.txt" | \
        while read sub; do
            dig +short $sub | grep -E '^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$'
        done | sort -u > "$OUTDIR/ports/ips.txt"

    local ip_count=$(wc -l < "$OUTDIR/ports/ips.txt")
    echo "[*] Scanning $ip_count unique IPs..."

    # Fast scan top ports
    nmap -iL "$OUTDIR/ports/ips.txt" \
        --top-ports 1000 \
        -T4 \
        --open \
        -oA "$OUTDIR/ports/nmap_top1000" \
        2>/dev/null

    # Extract open ports summary
    grep "open" "$OUTDIR/ports/nmap_top1000.nmap" | \
        grep -v "filtered" > "$OUTDIR/ports/open_ports.txt" 2>/dev/null || true

    echo -e "${GREEN}[+] Port scan complete${NC}"
}

screenshot() {
    echo -e "\n${YELLOW}[*] Phase 4: Screenshots${NC}"

    mkdir -p "$OUTDIR/screenshots"

    echo "[*] Taking screenshots..."
    gowitness file -f "$OUTDIR/alive/live_urls.txt" \
        --screenshot-path "$OUTDIR/screenshots" \
        --threads 10 \
        2>/dev/null

    echo -e "${GREEN}[+] Screenshots saved${NC}"
}

content_discovery() {
    echo -e "\n${YELLOW}[*] Phase 5: Content Discovery (Top 10 targets)${NC}"

    mkdir -p "$OUTDIR/content"

    # Only hit top 10 to avoid noise
    head -10 "$OUTDIR/alive/live_urls.txt" | while read url; do
        domain=$(echo $url | sed 's|https\?://||' | cut -d'/' -f1)
        echo "[*] Fuzzing: $url"

        feroxbuster -u "$url" \
            -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
            -t 20 \
            -k \
            --silent \
            -o "$OUTDIR/content/${domain}_dirs.txt" \
            2>/dev/null || true
    done

    echo -e "${GREEN}[+] Content discovery complete${NC}"
}

vuln_scan() {
    echo -e "\n${YELLOW}[*] Phase 6: Nuclei Vulnerability Scan${NC}"

    mkdir -p "$OUTDIR/vulns"

    echo "[*] Running nuclei..."
    nuclei -l "$OUTDIR/alive/live_urls.txt" \
        -severity low,medium,high,critical \
        -o "$OUTDIR/vulns/nuclei_results.txt" \
        -silent \
        2>/dev/null

    if [ -s "$OUTDIR/vulns/nuclei_results.txt" ]; then
        echo -e "${RED}[!] Vulnerabilities found! Check $OUTDIR/vulns/nuclei_results.txt${NC}"
    else
        echo -e "${GREEN}[+] No vulnerabilities detected${NC}"
    fi
}

generate_report() {
    echo -e "\n${YELLOW}[*] Generating Summary Report${NC}"

    cat << EOF > "$OUTDIR/REPORT.md"
# Recon Report: $TARGET
Generated: $(date)

## Summary
- Subdomains found: $(wc -l < "$OUTDIR/subdomains/all_subs.txt")
- Live URLs: $(wc -l < "$OUTDIR/alive/live_urls.txt")
- Unique IPs: $(wc -l < "$OUTDIR/ports/ips.txt")

## Live Hosts
\`\`\`
$(cat "$OUTDIR/alive/httpx_full.txt")
\`\`\`

## Open Ports
\`\`\`
$(cat "$OUTDIR/ports/open_ports.txt" 2>/dev/null || echo "See nmap output")
\`\`\`

## Vulnerabilities
\`\`\`
$(cat "$OUTDIR/vulns/nuclei_results.txt" 2>/dev/null || echo "None detected")
\`\`\`

## Files
- Subdomains: subdomains/all_subs.txt
- Live URLs: alive/live_urls.txt
- Screenshots: screenshots/
- Port Scans: ports/
- Content Discovery: content/
- Vulnerabilities: vulns/
EOF

    echo -e "${GREEN}[+] Report saved: $OUTDIR/REPORT.md${NC}"
}

# Main
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target.com>"
    exit 1
fi

banner
check_tools

mkdir -p "$OUTDIR"

subdomain_enum
resolve_alive
port_scan
screenshot
content_discovery
vuln_scan
generate_report

echo -e "\n${GREEN}================================================${NC}"
echo -e "${GREEN}  RECON COMPLETE${NC}"
echo -e "${GREEN}  Results: $OUTDIR${NC}"
echo -e "${GREEN}================================================${NC}"
```

## Subdomain Takeover Checker

Quick script to check for potential subdomain takeovers.

```bash
#!/bin/bash
# takeover_check.sh - Check subdomains for takeover potential

SUBS_FILE=$1

if [ -z "$SUBS_FILE" ]; then
    echo "Usage: $0 <subdomains.txt>"
    exit 1
fi

echo "[*] Checking for potential takeovers..."

# Known fingerprints
declare -A FINGERPRINTS=(
    ["There isn't a GitHub Pages site here"]="GitHub Pages"
    ["NoSuchBucket"]="AWS S3"
    ["The specified bucket does not exist"]="AWS S3"
    ["Fastly error: unknown domain"]="Fastly"
    ["The request could not be satisfied"]="CloudFront"
    ["Sorry, this shop is currently unavailable"]="Shopify"
    ["There's nothing here"]="Tumblr"
    ["Do you want to register"]="WordPress"
    ["Help Center Closed"]="Zendesk"
    ["This UserVoice subdomain is currently available"]="UserVoice"
)

while read subdomain; do
    # Get CNAME
    cname=$(dig +short CNAME "$subdomain" | head -1)

    if [ -n "$cname" ]; then
        # Check if CNAME points to known vulnerable services
        response=$(curl -s -L -m 10 "http://$subdomain" 2>/dev/null)

        for fingerprint in "${!FINGERPRINTS[@]}"; do
            if echo "$response" | grep -q "$fingerprint"; then
                echo "[!] POTENTIAL TAKEOVER: $subdomain"
                echo "    CNAME: $cname"
                echo "    Service: ${FINGERPRINTS[$fingerprint]}"
                echo ""
            fi
        done
    fi
done < "$SUBS_FILE"

echo "[*] Check complete"
```

## Tech Stack Fingerprinting

Identify technologies in use across discovered hosts.

```python
#!/usr/bin/env python3
"""
tech_fingerprint.py - Identify web technologies
"""

import requests
import re
import sys
from concurrent.futures import ThreadPoolExecutor, as_completed

# Technology signatures
SIGNATURES = {
    'WordPress': [
        r'/wp-content/',
        r'/wp-includes/',
        r'<meta name="generator" content="WordPress',
    ],
    'Drupal': [
        r'Drupal.settings',
        r'/sites/default/files',
        r'<meta name="Generator" content="Drupal',
    ],
    'Joomla': [
        r'/media/jui/',
        r'<meta name="generator" content="Joomla',
    ],
    'Laravel': [
        r'laravel_session',
        r'XSRF-TOKEN',
    ],
    'ASP.NET': [
        r'__VIEWSTATE',
        r'__EVENTVALIDATION',
        r'X-AspNet-Version',
    ],
    'React': [
        r'_reactRootContainer',
        r'data-reactroot',
    ],
    'Vue.js': [
        r'__vue__',
        r'v-cloak',
    ],
    'Angular': [
        r'ng-version',
        r'ng-app',
    ],
    'jQuery': [
        r'jquery[-.]?\d*\.?(min\.)?js',
    ],
    'Bootstrap': [
        r'bootstrap[-.]?\d*\.?(min\.)?css',
    ],
    'Nginx': [
        r'nginx',
    ],
    'Apache': [
        r'Apache',
    ],
    'IIS': [
        r'Microsoft-IIS',
    ],
    'Cloudflare': [
        r'cloudflare',
        r'cf-ray',
    ],
}


def fingerprint_url(url):
    """Check URL against known signatures"""
    try:
        resp = requests.get(url, timeout=10, verify=False, allow_redirects=True)
        headers = str(resp.headers).lower()
        body = resp.text

        found = []
        for tech, patterns in SIGNATURES.items():
            for pattern in patterns:
                if re.search(pattern, body, re.IGNORECASE) or \
                   re.search(pattern, headers, re.IGNORECASE):
                    found.append(tech)
                    break

        return url, list(set(found))

    except Exception as e:
        return url, []


def main():
    if len(sys.argv) < 2:
        print("Usage: python3 tech_fingerprint.py <urls.txt>")
        sys.exit(1)

    with open(sys.argv[1], 'r') as f:
        urls = [line.strip() for line in f if line.strip()]

    print(f"[*] Fingerprinting {len(urls)} URLs...\n")

    results = {}
    with ThreadPoolExecutor(max_workers=20) as executor:
        futures = {executor.submit(fingerprint_url, url): url for url in urls}

        for future in as_completed(futures):
            url, techs = future.result()
            if techs:
                results[url] = techs
                print(f"{url}")
                print(f"  -> {', '.join(techs)}\n")

    # Summary
    print("\n" + "="*50)
    print("TECHNOLOGY SUMMARY")
    print("="*50)

    tech_count = {}
    for url, techs in results.items():
        for tech in techs:
            tech_count[tech] = tech_count.get(tech, 0) + 1

    for tech, count in sorted(tech_count.items(), key=lambda x: x[1], reverse=True):
        print(f"{tech}: {count}")


if __name__ == '__main__':
    import urllib3
    urllib3.disable_warnings()
    main()
```

## Cloud Asset Discovery

Find cloud resources associated with a target.

```bash
#!/bin/bash
# cloud_enum.sh - Discover cloud assets

TARGET=$1
OUTDIR="./cloud_$TARGET"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <company-name>"
    exit 1
fi

mkdir -p "$OUTDIR"

echo "[*] Cloud enumeration for: $TARGET"

# S3 bucket permutations
echo "[*] Checking S3 buckets..."
PERMS=("$TARGET" "${TARGET}-dev" "${TARGET}-prod" "${TARGET}-staging" \
       "${TARGET}-backup" "${TARGET}-assets" "${TARGET}-static" \
       "${TARGET}-media" "${TARGET}-data" "${TARGET}-logs")

for bucket in "${PERMS[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" "https://${bucket}.s3.amazonaws.com")
    if [ "$status" != "404" ]; then
        echo "[+] S3: $bucket (Status: $status)" | tee -a "$OUTDIR/s3_buckets.txt"
    fi
done

# Azure blob storage
echo "[*] Checking Azure blobs..."
for name in "${PERMS[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" "https://${name}.blob.core.windows.net")
    if [ "$status" != "400" ] && [ "$status" != "000" ]; then
        echo "[+] Azure: $name (Status: $status)" | tee -a "$OUTDIR/azure_blobs.txt"
    fi
done

# GCP storage
echo "[*] Checking GCP buckets..."
for name in "${PERMS[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" "https://storage.googleapis.com/${name}")
    if [ "$status" != "404" ]; then
        echo "[+] GCP: $name (Status: $status)" | tee -a "$OUTDIR/gcp_buckets.txt"
    fi
done

echo "[*] Results saved in $OUTDIR/"
```

## JS File Analysis

Extract endpoints and secrets from JavaScript files.

```python
#!/usr/bin/env python3
"""
js_analyzer.py - Extract juicy info from JS files
"""

import requests
import re
import sys
from urllib.parse import urljoin

# Patterns to find
PATTERNS = {
    'api_endpoints': [
        r'["\']/(api|v\d)/[^"\']+["\']',
        r'["\']https?://[^"\']*api[^"\']*["\']',
    ],
    'aws_keys': [
        r'AKIA[0-9A-Z]{16}',
        r'["\']aws[_-]?(access|secret)[_-]?key["\']:\s*["\'][^"\']+["\']',
    ],
    'api_keys': [
        r'api[_-]?key["\']?\s*[:=]\s*["\'][a-zA-Z0-9_\-]{20,}["\']',
        r'authorization["\']?\s*[:=]\s*["\']Bearer\s+[^"\']+["\']',
    ],
    'tokens': [
        r'token["\']?\s*[:=]\s*["\'][a-zA-Z0-9_\-\.]{20,}["\']',
        r'jwt["\']?\s*[:=]\s*["\'][a-zA-Z0-9_\-\.]+["\']',
    ],
    'private_keys': [
        r'-----BEGIN (RSA |EC )?PRIVATE KEY-----',
    ],
    'internal_urls': [
        r'https?://(?:localhost|127\.0\.0\.1|10\.\d+\.\d+\.\d+|192\.168\.\d+\.\d+)[^\s"\']*',
    ],
    'emails': [
        r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}',
    ],
}


def analyze_js(url_or_content):
    """Analyze JavaScript for sensitive data"""

    if url_or_content.startswith('http'):
        try:
            resp = requests.get(url_or_content, timeout=10)
            content = resp.text
            source = url_or_content
        except:
            return None, {}
    else:
        content = url_or_content
        source = "inline"

    findings = {}

    for category, patterns in PATTERNS.items():
        matches = []
        for pattern in patterns:
            found = re.findall(pattern, content, re.IGNORECASE)
            matches.extend(found)

        if matches:
            findings[category] = list(set(matches))

    return source, findings


def extract_js_urls(html_content, base_url):
    """Extract JS file URLs from HTML"""
    pattern = r'<script[^>]+src=["\']([^"\']+\.js[^"\']*)["\']'
    matches = re.findall(pattern, html_content, re.IGNORECASE)

    urls = []
    for match in matches:
        if match.startswith('//'):
            urls.append('https:' + match)
        elif match.startswith('/'):
            urls.append(urljoin(base_url, match))
        elif match.startswith('http'):
            urls.append(match)
        else:
            urls.append(urljoin(base_url, match))

    return urls


def main():
    if len(sys.argv) < 2:
        print("Usage: python3 js_analyzer.py <url|urls.txt>")
        sys.exit(1)

    target = sys.argv[1]

    if target.startswith('http'):
        urls = [target]
    else:
        with open(target, 'r') as f:
            urls = [line.strip() for line in f if line.strip()]

    all_findings = {}

    for url in urls:
        print(f"\n[*] Analyzing: {url}")

        # Get main page and extract JS
        try:
            resp = requests.get(url, timeout=10)
            js_urls = extract_js_urls(resp.text, url)
            print(f"    Found {len(js_urls)} JS files")

            for js_url in js_urls:
                source, findings = analyze_js(js_url)
                if findings:
                    all_findings[source] = findings
                    print(f"\n    [!] {source}")
                    for cat, items in findings.items():
                        print(f"        {cat}:")
                        for item in items[:5]:  # First 5
                            print(f"          - {item[:80]}")

        except Exception as e:
            print(f"    Error: {e}")

    # Summary
    print("\n" + "="*50)
    print("FINDINGS SUMMARY")
    print("="*50)

    categories = {}
    for source, findings in all_findings.items():
        for cat in findings.keys():
            categories[cat] = categories.get(cat, 0) + len(findings[cat])

    for cat, count in sorted(categories.items(), key=lambda x: x[1], reverse=True):
        print(f"{cat}: {count} findings")


if __name__ == '__main__':
    main()
```

---

## Putting It Together

```bash
# Full recon workflow
./recon.sh target.com

# Check for takeovers
./takeover_check.sh ./recon_target.com_*/subdomains/all_subs.txt

# Tech stack
python3 tech_fingerprint.py ./recon_target.com_*/alive/live_urls.txt

# Cloud assets
./cloud_enum.sh targetcompany

# JS analysis
python3 js_analyzer.py ./recon_target.com_*/alive/live_urls.txt
```

These scripts have found me countless bugs and entry points. Customize the wordlists, adjust the thread counts for your setup, and add your own signature patterns as you discover new technologies.

Recon isn't glamorous, but it's where engagements are won.
