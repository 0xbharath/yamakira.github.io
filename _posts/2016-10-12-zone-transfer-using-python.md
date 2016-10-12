---
layout:            post
title:             "DNS zone transfer using Python"
menutitle:         "DNS zone transfer using Python"
date:              2016-10-12 00:40:00 +0300
tags:              python dns pentesting
category:          Python
author:            bharath
cover:             /assets/header_bg.jpg
published:         true
cover:             /assets/header_bg.jpg
language:          EN
comments:          true
---

It is important to understand some key terms regarding Domain Name System(DNS) before trying to understand DNS zone transfer.


**Zone** <br>
- Think of zone as a “domain”: a collection of hostnames/IP pairs all managed together. <br>
- Sub-domains are sometimes part of the main zone, sometimes they are a separate zone.

<IMG class="displayed" align="center" STYLE="WIDTH:300px; HEIGHT:300px" src="{{ site.github.url }}/media/img/dns_zone.gif" alt="DNS zone">

**Nameserver** <br>
- This is server software that answers DNS questions.  <br>
- Sometimes a nameserver knows the answer directly (if it\'s “authoritative” for the zone), other times it has to go out to the internet and ask around to find the answer (if it\'s a recursive nameserver).  <br>
- There is wide variety of software that performs this service: BIND, PowerDNS etc.

**Authoritative Nameserver** <br>
- For every zone, somebody has to maintain a file of the hostnames and IP address associations. This is generally an administrative function performed by a human, and in most cases one machine has this file. It\'s the zone master.  <br>
- **Zones with multiple public nameservers make administrative arrangements to transfer the zone data automatically to additional slave nameservers**, all of which are authoritative as far as the outside world is concerned.

More info on DNS protocol: <br>
[An Illustrated Guide to the Kaminsky DNS Vulnerability](http://www.unixwiz.net/techtips/iguide-kaminsky-dns-vuln.html) <br>

###### What is DNS zone transfer?

DNS zone transfer itself is not an attack, zone transfer is a type of DNS transaction where a DNS server passes a copy of part of it\'s database(zone file) to another DNS server. It is a mechanism available to replicate/update DNS databases across a set of DNS servers. <br>

DNS zone transfer is always initiated by client/slave by inducing DNS query type **AXFR**. Zone transfers may be performed using two methods, full AXFR and incremental IXFR.

A zone transfer uses the TCP  for transport with the notion of client–server transaction. The client requesting a zone transfer may be a slave server, requesting data from a master server. The portion of the database that is replicated is a zone.

<IMG class="displayed" align="center" STYLE="WIDTH:440px; HEIGHT:250px" src="{{ site.github.url }}/media/img/zone_transfer.gif" alt="zone transfer">

More info on DNS zone transfer: <br>
[DNS zone trasfer - wikipedia](https://en.wikipedia.org/wiki/DNS_zone_transfer) <br>
[RFC 5936 - DNS Zone Transfer Protocol (AXFR)](https://tools.ietf.org/html/rfc5936)


###### Zone transfer - security implications

The data contained in a DNS zone may be sensitive from a security aspect(details such as name Servers, host names, MX records, CNAME records, glue records (delegation for child Domains), zone serial number, Time To Live etc.

From an attacker perspective, a zone file contains treasure trove of information regarding the target network. DNS zone may reveal a lot of topological information about internal network.

Having list of all the sub-domains that are part of a domain will make work of a vulnerability hunter easy.


###### Zone transfer attack

In a zone transfer attack, an attacker acts like a slave server and requests the master for a copy of the zone records. If the DNS server responds with requested records, your attack is successful.

Although most DNS servers these days deny zone transfers made by unauthorized/external clients, it\'s not hard to find vulnerable DNS servers esp. DNS servers maintained by organizations themselves as part of the internal networks.

To perform a zone transfer you need the following details: <br>
- A domain that you want to gather information about. <br>
- A zone master/primary DNS server that holds the zone information for a domain.

<div class="tip">
The legality of performing zone transfer on domains you don\'t own is uncertain and is out right illegal in some jurisdictions. <br>
Perform zone transfers at your own risk. Obtain permission before performing zone transfer to play safe. <br>
</div>

If you want to practice with out getting into trouble, there is a deliberately vulnerable nameserver out there for the exact purpose: <br>
[ZoneTransfer.me](https://digi.ninja/projects/zonetransferme.php) <br>
**Nameserver**: nsztm1.digi.ninja <br>
**Domain**: zonetransfer.me

###### Zone transfer attack using dig

- dig is a very handy DNS utility available on most linux machines. 
- Let\'s perform a simple zone transfer using dig utility.
- Let\'s say we want to find the sub-domains of `iitk.ac.in`, we need to find the nameservers for a target domain(NS records).

```bash
verax@untamed $ dig +nocmd +nocomments NS iitk.ac.in
;iitk.ac.in.			IN	NS
iitk.ac.in.		43200	IN	NS	proxy.iitk.ac.in.
iitk.ac.in.		43200	IN	NS	ns1.iitk.ac.in.
iitk.ac.in.		43200	IN	NS	ns2.iitk.ac.in.
```

- Let\'s initiate a zone transfer with the nameserver `ns1.iitk.ac.in.`, for domain `iitk.ac.in`. As a result of the query we retrived 166 records, with careful analysis it\'s trivial to map the internal network. Another interesting note is that SOA record is retrived before initiating a zone transfer.

```bash
verax@untamed ~/$ dig +nocmd +nocomments AXFR @ns1.iitk.ac.in. iitk.ac.in
iitk.ac.in.		43200	IN	SOA	ns1.iitk.ac.in. root.ns1.iitk.ac.in. 201609222 10800 3600 1209600 43200
iitk.ac.in.		43200	IN	NS	ns1.iitk.ac.in.
iitk.ac.in.		43200	IN	NS	ns2.iitk.ac.in.
iitk.ac.in.		43200	IN	NS	proxy.iitk.ac.in.
home.iitk.ac.in.	43200	IN	A	202.3.77.174
m3cloud.iitk.ac.in.	43200	IN	A	103.246.106.161
mail.iitk.ac.in.	43200	IN	A	202.3.77.162
[... snipped ...]
mail4.iitk.ac.in.	43200	IN	A	202.3.77.189
webmail.iitk.ac.in.	43200	IN	A	202.3.77.185
www.webmap.iitk.ac.in.	43200	IN	A	202.3.77.74
wiki.iitk.ac.in.	43200	IN	A	103.246.106.116
www.iitk.ac.in.		43200	IN	A	202.3.77.184
iitk.ac.in.		43200	IN	SOA	ns1.iitk.ac.in. root.ns1.iitk.ac.in. 201609222 10800 3600 1209600 43200
;; Query time: 219 msec
;; SERVER: 202.3.77.171#53(202.3.77.171)
;; WHEN: Wed Oct 12 18:38:01 IST 2016
;; XFR size: 166 records (messages 1, bytes 3769)
```

###### Zone transfer attack using Python

- Python does\'nt have an inbuit DNS module. We\'ll use `dnspython`, a third party module/toolkit for DNS operations in Python.
- Let\'s find out the nameservers for the domain `iitk.ac.in` using dnspython. I\'ll go ahead and retrive all the common DNS records for the domain.
  - `dns.resolver` is the client part of dnspython.
  - `dns.resolver.query` is how you make a query. provide domain name, query type as arguments.

```python
import argparse, dns.resolver

def lookup(name):
    print name
    for qtype in 'A', 'AAAA', 'MX', 'TXT', 'SOA':
        answer = dns.resolver.query(name,qtype, raise_on_no_answer=False)
        if answer.rrset is not None:
            print(answer.rrset)

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Resolve a name using DNS')
    parser.add_argument('name', help='name that you want to look up in DNS')
    lookup(parser.parse_args().name)
```

```bash
verax@untamed ~/dns_scripts $ python dns_queries.py iitk.ac.in
iitk.ac.in
iitk.ac.in. 37799 IN A 202.3.77.184
iitk.ac.in. 43180 IN MX 10 mail0.iitk.ac.in.
iitk.ac.in. 43180 IN MX 10 mail1.iitk.ac.in.
iitk.ac.in. 43200 IN NS proxy.iitk.ac.in.
iitk.ac.in. 43200 IN NS ns2.iitk.ac.in.
iitk.ac.in. 43200 IN NS ns1.iitk.ac.in.
iitk.ac.in. 43181 IN SOA ns1.iitk.ac.in. root.ns1.iitk.ac.in. 201609222 10800 3600 1209600 43200
```

- Let's initiate a zone transfer with the nameserver `ns1.iitk.ac.in.`, for domain `iitk.ac.in` using dnspython.
  - `dns.query`supports XFR query i.e. zone transfer.
  - `dns.zone` helps use manage the zone details
  -  we are trying to extract only the sub-domain details from the zone details.

```python
import dns.query
import dns.zone

z = dns.zone.from_xfr(dns.query.xfr('ns1.iitk.ac.in', 'iitk.ac.in'))
names = z.nodes.keys()
names.sort()
for n in names:
    print z[n].to_text(n)
```
- In this example, it\'s very clear that first the SOA records are transferred.
- All the nameservers, mail servers are represented with their hostnames, rather than FQDN.

```bash
verax@untamed ~ $ python zone_transfer.py 
@ 43200 IN SOA ns1 root.ns1 201609222 10800 3600 1209600 43200
@ 43200 IN NS ns1
@ 43200 IN NS ns2
@ 43200 IN NS proxy
@ 43200 IN A 202.3.77.184
@ 43200 IN MX 10 mail0
@ 43200 IN MX 10 mail1
agrotagger 43200 IN CNAME m3cloud
all-iits 43200 IN A 202.3.77.160
alumni 43200 IN A 202.3.77.176
antaragni 43200 IN CNAME students
appsgate 43200 IN A 202.3.77.165
aqi 43200 IN A 103.246.106.117
[... snipped ...]
```
<div class="tip">
DO NOT assume that all the records that are part of zone transfer must be part of the organization/domain that you are targeting. <br>
There is nothing stopping the administrator from adding additional records from anywhere on the Internet to this zone. <br>
Be cautious, if you are pentesting, check all the records using DNS-reverse resolution, traceroute and whois to determine the ownership.
</div>


###### Securing DNS zone transfer

The default behavior for DNS zone transfer permits any host to request and receive a full zone transfer for a Domain. The consequences of revealing zone details to any potential attacker could be devastating.

Two methods to secure zone transfers are BIND DNS access control list and DNS Transaction Signatures (TSIG). I'll not delve into the details of securing DNS zone transfer in this post but readers can explore on their own.


More about securing zone transfer: <br>
[Why securing zone transfer is necessary?](https://www.sans.org/reading-room/whitepapers/dns/securing-dns-zone-transfer-868) [PDF] <br>

<br>


###### References

[DNS zone trasfer - wikipedia](https://en.wikipedia.org/wiki/DNS_zone_transfer) <br>
[dnspython docs - examples](http://www.dnspython.org/examples.html) <br>
[An Illustrated Guide to the Kaminsky DNS Vulnerability](http://www.unixwiz.net/techtips/iguide-kaminsky-dns-vuln.html) <br>
[ZoneTransfer.me](https://digi.ninja/projects/zonetransferme.php) <br>
[Why securing zone transfer is necessary?](https://www.sans.org/reading-room/whitepapers/dns/securing-dns-zone-transfer-868) [PDF] <br>
Image courtesy: [Name Servers and Zones](http://docstore.mik.ua/orelly/networking_2ndEd/dns/ch02_04.htm) <br>
Image courtesy: [DNS & BIND - Chap 10](http://www.diablotin.com/librairie/networking/dnsbind/ch10_02.htm) <br>
