---
# Title, summary, and page position.
linktitle: Security Correlation
summary: Using information from logs to identify the details of an attack.
weight: 2
icon: book
icon_pack: fas

menu:
  docs:
    name: Security Correlation
    weight: 2

# Page metadata.
title: Security Correlation
type: book  # Do not modify.
---

Before reading this page, it will be useful to read [this document](https://www.sans.org/reading-room/whitepapers/bestprac/effective-case-modeling-security-information-event-management-33319).

We will look at some fake example logs from different services, and see how we can interpret the raw data to identify attack attempts.

This page is largely adapted from a presentation given to us in a lecture by AARNet.

## Scenario

An attacker with IP x.x.x.x is running techniques on a server. The packets go through a router, firewall, and a network intrusion detection system (IDS) before reaching application servers.

## Router

Suppose the router access control lists allow all ingress TCP packets to port 80 at y.y.y.3 because this is where the web server is.

```
30/08/2020 10:30:45 router.example.company 195768: 30/08/2020 10:30:44 %SEC-6-IPACCESSLOGP: list from-internet denied tcp x.x.x.x.(1546) -> y.y.y.1(80), 1 packet
30/08/2020 10:30:48 router.example.company 195770: 30/08/2020 10:30:48 %SEC-6-IPACCESSLOGP: list from-internet denied tcp x.x.x.x.(1546) -> y.y.y.2(80), 1 packet
30/08/2020 10:30:51 router.example.company 195774: 30/08/2020 10:30:50 %SEC-6-IPACCESSLOGP: list from-internet denied tcp x.x.x.x.(1546) -> y.y.y.4(80), 1 packet
30/08/2020 10:30:56 router.example.company 195780: 30/08/2020 10:30:55 %SEC-6-IPACCESSLOGP: list from-internet denied tcp x.x.x.x.(1546) -> y.y.y.5(80), 1 packet
30/08/2020 10:30:57 router.example.company 195781: 30/08/2020 10:30:57 %SEC-6-IPACCESSLOGP: list from-internet denied tcp x.x.x.x.(1546) -> y.y.y.6(80), 1 packet
```

From these logs we see that the target IP is increasing, indicating that the attacker is doing a broad scan of the network. Notice that y.y.y.3 isn't in the logs. This is likely because the packet was allowed through to the web server, so the attacker probably found y.y.y.3.

Now from these logs, a way to present data effectively is to answer the five 'W's.
- Who\
x.x.x.x
- What\
Scan of y.y.y.0/24 for open servers at port 80. Likely found that y.y.y.3 was open
- When\
30 August 2020, between 10:30:45 and 10:30:57
- Where\
Company [DMZ](https://en.wikipedia.org/wiki/DMZ_(computing)) network
- Why\
Likely some kind of reconnaissance, looking for possible attack surface

## Firewall

```
31/08/2020 08:23:34 firewall.example.company http-gw\[12345\]: log host=nodnsquery/x.x.x.x protocol=http cmd=get dest=y.y.y.y.3 path=/cgi-bin/phf ID=11111111111
31/08/2020 08:23:38 firewall.example.company http-gw\[12345\]: log host=nodnsquery/x.x.x.x protocol=http cmd=get dest=y.y.y.y.3 path=/cgi-bin/formmail ID=11111111112
31/08/2020 08:23:42 firewall.example.company http-gw\[12345\]: log host=nodnsquery/x.x.x.x protocol=http cmd=get dest=y.y.y.y.3 path=/cgi-bin/survey.cgi ID=11111111113
```

The attacker connected to the web server throught HTTP multiple times. The logs don't tell us anything about whether the attempts were successful or not.

Looking at the paths they tried, we can search up what they tried to access for vulnerabilities.
- [phf](http://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=phf)
- [formmail](http://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=formmail)
- [survey.cgi](http://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=survey.cgi)

As these attempts were conducted in succession, the attacker may have been looking for vulnerabilities of the same type. Then the likely candidates are the remote execution CVEs.
- phf\
[CVE-1999-0067](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-0067)
- formmail\
[CVE-1999-0172](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-0172)
- survey.cgi\
[CVE-1999-0936](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-0936)

Again we try to answer the five 'W's.
- Who\
x.x.x.x
- What\
Three HTTP connections to y.y.y.3 (an open web server). Attempts to access the CGI scripts `phf`, `formmail`, and `survey.cgi`. Unknown if successful. No normal web activity from this host so it seems that they are purely attempting malicious activity.
- When\
31 August 2020 at 08:23:34
- Where\
Company DMZ network
- Why\
The CGI scripts all have remote execution CVEs. As there is no normal browsing activity from this host, these access attempts suggest malicious activity.

## IDS

```
[**][1:886:3] WEB-CGI phf access [**]
[Classification: Attempted Information Leak][Priority: 2]
31/08/2020 08:23:34.435267 x.x.x.x:1234 -> y.y.y.3:80
TCP TTL:52 TOS:0x0 ID:61884 IpLen:20 DgmLen:280 DF
***AP*** Seq: 0x591AF831 Ack: 0x92D23FAF Win: 0x16D0 TcpLen: 32
TCP Options (3) => NOP NOP TS: 59902357 300726
[Xref => http://www.securityfocus.com/bid/629]
[Xref => http://www.whitehats.com/info/IDS128]
[Xref => http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE1999-0067]

[**][1:884:2] WEB-CGI formmail access [**]
[Classification: Attempted Information Leak][Priority: 2]
31/08/2020 08:23:38.325838 x.x.x.x:1235 -> y.y.y.3:80
TCP TTL:52 TOS:0x0 ID:15383 IpLen:20 DgmLen:285 DF
***AP*** Seq: 0x85C51FDB Ack: 0xC0D4B803 Win: 0x16D0 TcpLen: 32
TCP Options (3) => NOP NOP TS: 59974615 372988
[Xref => http://www.securityfocus.com/bid/1187]
[Xref => http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-0172]
[Xref => http://www.whitehats.com/info/IDS226]

[**][1:871:2] WEB-CGI survey.cgi access [**]
[Classification: Attempted Information Leak][Priority: 2]
31/08/2020 08:23:42.584739 x.x.x.x:1236 -> y.y.y.3:80
TCP TTL:52 TOS:0x0 ID:32890 IpLen:20 DgmLen:295 DF
***AP*** Seq: 0x8B55C63C Ack: 0xC624745D Win: 0x16D0 TcpLen: 32
TCP Options (3) => NOP NOP TS: 59983434 381809
[Xref => http://www.securityfocus.com/bid/1817]
[Xref => http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-0936]
```

These alerts were triggered by the following rules:

```
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"WEB-CGI phf access";flags: A+; uricontent:"/phf"; nocase; reference:bugtraq,629; reference:arachnids,128; reference:cve,CVE-1999-0067; classtype:attempted-recon; sid:886; rev:3;)
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"WEB-CGI formmail access";flags: A+; uricontent:"/formmail"; nocase; reference:bugtraq,1187; reference:cve,CVE-1999-0172; reference:arachnids,226; classtype:attempted-recon; sid:884; rev:2;)
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"WEB-CGI survey.cgi access";flags: A+; uricontent:"/survey.cgi"; nocase; reference:bugtraq,1817; reference:cve,CVE-1999-0936; classtype:attempted-recon; sid:871; rev:2;)
```

The rules trigger the alerts when there are specific strings in the URL. There may be false positives so these are reviewed manually. It is likely that these attempts are malicious because they are so close together in time from the same host. We aren't told if the attempts were successful.

Again we try to answer the five 'W's.
- Who\
x.x.x.x
- What\
Three HTTP connections to y.y.y.3 attempting to access the CGI scripts `phf`, `formmail`, and `survey.cgi`. We don't have any additional logs to know for sure if this is a false positive, but it is unlikely a non-malicious user would attempt to access these three scripts consecutively with incrementing source port.
- When\
31 August 2020 at 08:23:34
- Where\
Company DMZ network
- Why\
If these vulnerable scripts are on the web server, the attacker could perform remote command execution.

## Web Server

access_log
```
x.x.x.x -- [31/08/2020 08:23:34 -0400]"GET /cgi-bin/phf HTTP/1.0" 404 304 "." "Lynx/2.8.5dev.2 libwww-FM/2.14 SSL-MM/1.4.1 OpenSSL/0.9.6a"
x.x.x.x -- [31/08/2020 08:23:38 -0400]"GET /cgi-bin/formmail HTTP/1.0" 404 309 "." "Lynx/2.8.5dev.2 libwww-FM/2.14 SSL-MM/1.4.1 OpenSSL/0.9.6a"
x.x.x.x -- [31/08/2020 08:23:42 -0400]"GET /cgi-bin/survey.cgi HTTP/1.0" 404 311 "." "Lynx/2.8.5dev.2 libwww-FM/2.14 SSL-MM/1.4.1 OpenSSL/0.9.6a"
```

error_log
```
[31/08/2020 08:23:34][error][client x.x.x.x] script not found or unable to stat: /var/www/cgi-bin/phf
[31/08/2020 08:23:38][error][client x.x.x.x] script not found or unable to stat: /var/www/cgi-bin/formmail
[31/08/2020 08:23:42][error][client x.x.x.x] script not found or unable to stat: /var/www/cgi-bin/survey.cgi
```

The `access_log` shows the access attempts and the `error_log` shows that the attempts failed. There are no other connections by the client suggesting the client is not a normal user.

Again we try to answer the five 'W's.
- Who\
x.x.x.x\
Likely to be a Unix or Linux host using Lynx v.2.8.5dev2
- What\
Three HTTP connections to y.y.y.3 (open web server), attempting to access the CGI scripts `phf`, `formmail`, and `survey.cgi`. The scripts were not found on the server so could not be accessed. As there were no other logs for the host, it appears this activity is not part of normal browsing.
- When\
31 August 2020 at 08:23:34
- Where\
Company DMZ network
- Why\
If these vulnerable scripts are on the web server, the attacker could perform remote command execution.

## Conclusion

Each log type individually didn't give us much information, but if we combine them all we get a clearer picture of the activity of x.x.x.x.

On 30 August 2020, host x.x.x.x scanned the y.y.y.0/24 network in search of web servers on port 80. Only the router could see this activity because the router denied the packets to the IPs that didn't contain the web server, so devices further along didn't see the activity at all.

On 31 August 2020, host x.x.x.x attempted three HTTP connections to the company web server at y.y.y.3. The firewall and web server `access_log` show that these were the only connection attempts by the host. These were attempts to access the CGI scripts `phf`, `formmail`, and `survey.cgi`. The firewall, web server `access_log`, and network IDS all confirm this. The web server `error_log` show that these connection attempts failed because the scripts were not on the server. So it is unlikely that there is any danger to the system from these attempts.