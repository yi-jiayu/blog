---
title: "Quick URL to ASN lookups"
subtitle: "Querying ASN information through URLs instead of IP addresses"
date: 2018-10-02T12:43:17+05:30
---

I'm often curious about how various websites are hosted. One piece of information which can give you a clue is the [autonomous system (AS)](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)) which the IP of the website falls under. We can use a combination of tools to find this.

With `dig`, we can lookup the IP address for a DNS name like `blog.jiayu.co`:

```console
$ dig +noall +answer blog.jiayu.co
blog.jiayu.co.		370	IN	CNAME	zealous-perlman-cb8cb5.netlify.com.
zealous-perlman-cb8cb5.netlify.com. 6 IN A	178.128.123.58
```

It shows that my blog is located at `206.189.89.118`.

With an IP address, we can find the corresponding AS information with an IP to ASP mapping service such as this one:

{{< linkpreview title="IP-to-ASN - Team Cymru" description="We exist to enable organizations to efficiently identify and eradicate threats within their network by providing unique and easily assessable insight that saves and improves human lives." url="https://www.team-cymru.com/IP-ASN-mapping.html" >}}

There's a web interface you can use [here](https://asn.cymru.com/):

{{< figure src="cymru-ip-to-asn.png" alt="Team Cymru IP to ASN Lookup website" >}}

Alternatively, you can use the WHOIS or DNS protocols:

```console
$ whois -h whois.cymru.com " -v 206.189.89.118"
AS      | IP               | BGP Prefix          | CC | Registry | Allocated  | AS Name
14061   | 206.189.89.118   | 206.189.80.0/20     | US | arin     | 1995-11-15 | DIGITALOCEAN-ASN - DigitalOcean, LLC, US
```

```console
$ dig +short 118.89.189.206.origin.asn.cymru.com TXT
"14061 | 206.189.80.0/20 | US | arin | 1995-11-15"
$ dig +short AS14061.asn.cymru.com TXT
"14061 | US | arin | 2012-09-25 | DIGITALOCEAN-ASN - DigitalOcean, LLC, US"
```

Whichever method we use, we find out that my blog is being served from a [DigitalOcean](https://www.digitalocean.com/) IP address. Does this mean that I'm running it off a personal VPS?

Nope, it's actually hosted by [Netlify](https://www.netlify.com/). But with this information, we can guess that Netlify uses DigitalOcean servers for some of its traffic. Not unexpected, since they just wrote about how they [migrated to a fully multi-cloud architecture](https://www.netlify.com/blog/2018/05/14/how-netlify-migrated-to-a-fully-multi-cloud-infrastructure/) earlier this year.

## Skipping steps

### Bash function

To find the AS for a URL, we needed to map the URL to an IP address and then map the IP address to an AS in two separate steps. To streamline this process, I wrote a bash function to quickly find the current AS for a URL:

```bash
asn() {
  if [ -z "$1" ]
  then
    echo "usage: asn url" 1>&2
    return 1
  fi

  ips=$(dig $DIG_ARGS +short $1)
  if [ -z "$ips" ]
  then
    echo "dig $1 failed" 1>&2
    return 1
  fi
  
  # convert ips to array
  iparr=($ips)
  
  # if there is more than one ip address,
  # use the bulk netcat interface
  # else use the whois interface
  if (( ${#iparr[*]} > 1 ))
  then
    printf "begin\nverbose\n$ips\nend\n" | netcat whois.cymru.com 43
  else
    whois -h whois.cymru.com " -v $ips"
  fi
}
```

Since some URLs have more than one A or AAAA record, we might need to make multiple IP to AS lookups per DNS lookup. In such cases, the function follows the guidelines in the [IP-to-ASN mapping documentation](https://www.team-cymru.com/IP-ASN-mapping.html#whois) and uses the bulk netcat interface:

> When issuing requests for two or more IPs we strongly suggest you use netcat for BULK IP submissions

> IPs that are seen abusing the whois server with large numbers of individual queries instead of using the bulk netcat interface will be null routed.

> The netcat interface should be used for groups of IP lists at a time in one single TCP query.

Note that this requires GNU `netcat`, which may or may not be installed on your system, to work.

### Usage

```console
$ asn github.com
Bulk mode; whois.cymru.com [2018-09-29 04:12:30 +0000]
36459   | 192.30.253.113   | 192.30.253.0/24     | US | arin     | 2012-11-15 | GITHUB - GitHub, Inc., US
36459   | 192.30.253.112   | 192.30.253.0/24     | US | arin     | 2012-11-15 | GITHUB - GitHub, Inc., US
$ asn google.com
AS      | IP               | BGP Prefix          | CC | Registry | Allocated  | AS Name
15169   | 216.58.199.142   | 216.58.199.0/24     | US | arin     | 2012-01-27 | GOOGLE - Google LLC, US
```

## Exploring further

It's interesting to see which AS various websites fall under.

Big companies tend to have their own AS:

```console
$ asn google.com
AS      | IP               | BGP Prefix          | CC | Registry | Allocated  | AS Name
15169   | 216.58.199.142   | 216.58.199.0/24     | US | arin     | 2012-01-27 | GOOGLE - Google LLC, US
$ asn facebook.com
AS      | IP               | BGP Prefix          | CC | Registry | Allocated  | AS Name
32934   | 157.240.16.35    | 157.240.0.0/17      | US | arin     | 2015-05-14 | FACEBOOK - Facebook, Inc., US
$ asn twitter.com
Bulk mode; whois.cymru.com [2018-10-02 04:50:46 +0000]
13414   | 104.244.42.1     | 104.244.42.0/24     | US | arin     | 2014-12-08 | TWITTER - Twitter Inc., US
13414   | 104.244.42.65    | 104.244.42.0/24     | US | arin     | 2014-12-08 | TWITTER - Twitter Inc., US
$ asn apple.com
Bulk mode; whois.cymru.com [2018-10-02 04:50:53 +0000]
714     | 17.178.96.59     | 17.178.0.0/15       | US | arin     | 1990-04-16 | APPLE-ENGINEERING - Apple Inc., US
714     | 17.142.160.59    | 17.142.0.0/15       | US | arin     | 1990-04-16 | APPLE-ENGINEERING - Apple Inc., US
714     | 17.172.224.47    | 17.168.0.0/13       | US | arin     | 1990-04-16 | APPLE-ENGINEERING - Apple Inc., US
$ asn amazon.com
Bulk mode; whois.cymru.com [2018-10-02 04:51:03 +0000]
16509   | 176.32.103.205   | 176.32.96.0/21      | IE | ripencc  | 2011-05-23 | AMAZON-02 - Amazon.com, Inc., US
16509   | 205.251.242.103  | 205.251.240.0/22    | US | arin     | 2010-08-27 | AMAZON-02 - Amazon.com, Inc., US
16509   | 176.32.98.166    | 176.32.98.0/24      | IE | ripencc  | 2011-05-23 | AMAZON-02 - Amazon.com, Inc., US
$ asn microsoft.com
Bulk mode; whois.cymru.com [2018-10-02 04:51:11 +0000]
8075    | 23.96.52.53      | 23.96.0.0/14        | US | arin     | 2013-06-18 | MICROSOFT-CORP-MSN-AS-BLOCK - Microsoft Corporation, US
...
8075    | 23.100.122.175   | 23.100.0.0/15       | US | arin     | 2013-06-18 | MICROSOFT-CORP-MSN-AS-BLOCK - Microsoft Corporation, US
```

Companies using AWS and GCP:

```console
$ asn netflix.com
Bulk mode; whois.cymru.com [2018-10-02 04:51:16 +0000]
16509   | 52.49.219.75     | 52.48.0.0/14        | US | arin     | 2015-09-02 | AMAZON-02 - Amazon.com, Inc., US
...
16509   | 52.30.45.198     | 52.30.0.0/15        | US | arin     | 1991-12-19 | AMAZON-02 - Amazon.com, Inc., US
$ asn snapchat.com
Bulk mode; whois.cymru.com [2018-10-02 04:53:18 +0000]
15169   | 216.239.34.21    | 216.239.34.0/24     | US | arin     | 2000-11-22 | GOOGLE - Google LLC, US
...
15169   | 216.239.38.21    | 216.239.38.0/24     | US | arin     | 2000-11-22 | GOOGLE - Google LLC, US
$ asn spotify.com
Bulk mode; whois.cymru.com [2018-10-02 04:53:40 +0000]
15169   | 104.199.240.211  | 104.199.224.0/19    | US | arin     | 2014-08-27 | GOOGLE - Google LLC, US
15169   | 104.199.64.136   | 104.199.64.0/19     | US | arin     | 2014-08-27 | GOOGLE - Google LLC, US
15169   | 104.154.127.47   | 104.154.96.0/19     | US | arin     | 2014-07-09 | GOOGLE - Google LLC, US
$ asn airbrb.com
Bulk mode; whois.cymru.com [2018-10-02 05:17:02 +0000]
14618   | 35.173.76.77     | 35.168.0.0/13       | US | arin     | 2016-08-09 | AMAZON-AES - Amazon.com, Inc., US
14618   | 34.237.227.65    | 34.224.0.0/12       | US | arin     | 2016-09-12 | AMAZON-AES - Amazon.com, Inc., US
```

Many websites are served from CDNs such as CloudFlare or Akamai as well:

```console
$ asn www.apple.com
Bulk mode; whois.cymru.com [2018-10-02 05:07:46 +0000]
Error: no ASN or IP match on line 3.
Error: no ASN or IP match on line 4.
Error: no ASN or IP match on line 5.
20940   | 184.28.35.18     | 184.28.34.0/23      | US | arin     | 2010-10-11 | AKAMAI-ASN1, US
$ asn www.airbnb.com
Bulk mode; whois.cymru.com [2018-10-02 05:16:36 +0000]
Error: no ASN or IP match on line 3.
...
Error: no ASN or IP match on line 6.
16625   | 103.51.152.47    | 103.51.152.0/24     | IN | apnic    | 2015-02-25 | AKAMAI-AS - Akamai Technologies, Inc., US
$ asn www.troyhunt.com
asn www.reddit.com
Bulk mode; whois.cymru.com [2018-10-02 05:19:41 +0000]
Error: no ASN or IP match on line 3.
13335   | 104.18.130.189   | 104.18.128.0/20     | US | arin     | 2014-03-28 | CLOUDFLARENET - Cloudflare, Inc., US
...
13335   | 104.18.128.189   | 104.18.128.0/20     | US | arin     | 2014-03-28 | CLOUDFLARENET - Cloudflare, Inc., US
yijiayu@cradily:~$ asn www.reddit.com
Bulk mode; whois.cymru.com [2018-10-02 05:19:48 +0000]
Error: no ASN or IP match on line 3.
54113   | 151.101.37.140   | 151.101.36.0/22     | US | arin     | 2016-02-01 | FASTLY - Fastly, US]
```

(The errors are due to `dig` returning URLs when a domain has `CNAME` records, and these aren't IP addresses hence "no ASN or IP match".)

It's important to remember that the ubiquitous `wwww` in front of many websites is a subdomain just like `blog` in `blog.jiayu.co`. The `www` version of a URL can have its own `A` and `AAAA` records and resolve to an entirely different IP from the non-`www` version. Usually, but not always, the `www` version of a URL is used for the website you visit in a browser. This is why we get different results for `apple.com` and `www.apple.com`, and `airbnb.com` and `www.airbnb.com`.

Finally, an interesting side-effect of the role of location in DNS queries:

```console
$ asn www.gov.sg
AS      | IP               | BGP Prefix          | CC | Registry | Allocated  | AS Name
9498    | 23.35.1.75       | 23.35.0.0/20        | US | arin     | 2011-05-16 | BBIL-AP BHARTI Airtel Ltd., IN
```

Oh? Is the Singapore government website is served from an Indian ISP?

```console
$ dig www.gov.sg

; <<>> DiG 9.10.6 <<>> +tcp www.gov.sg
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52913
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 8, ADDITIONAL: 11

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1280
;; QUESTION SECTION:
;www.gov.sg.			IN	A

;; ANSWER SECTION:
www.gov.sg.		20	IN	A	23.35.1.75

;; AUTHORITY SECTION:
www.gov.sg.		1079	IN	NS	eur5.akam.net.
www.gov.sg.		1079	IN	NS	usc3.akam.net.
www.gov.sg.		1079	IN	NS	ns1-188.akam.net.
www.gov.sg.		1079	IN	NS	use1.akam.net.
www.gov.sg.		1079	IN	NS	asia4.akam.net.
www.gov.sg.		1079	IN	NS	usw6.akam.net.
www.gov.sg.		1079	IN	NS	ns1-233.akam.net.
www.gov.sg.		1079	IN	NS	usc4.akam.net.

;; ADDITIONAL SECTION:
use1.akam.net.		23416	IN	A	72.246.46.64
ns1-233.akam.net.	23445	IN	A	193.108.91.233
ns1-233.akam.net.	23445	IN	AAAA	2600:1401:2::e9
eur5.akam.net.		23414	IN	A	23.74.25.64
usw6.akam.net.		32331	IN	A	23.61.199.64
usc4.akam.net.		23414	IN	A	184.26.160.65
asia4.akam.net.		23424	IN	A	184.85.248.64
usc3.akam.net.		23414	IN	A	96.7.50.64
ns1-188.akam.net.	100089	IN	A	193.108.91.188
ns1-188.akam.net.	100089	IN	AAAA	2600:1401:2::bc

;; Query time: 348 msec
;; SERVER: 10.10.10.1#53(10.10.10.1)
;; WHEN: Tue Oct 02 10:57:00 IST 2018
;; MSG SIZE  rcvd: 406
```

Turns out that it's also behind Akamai, and I was probably redirected to a closer edge node since I happened to be in India.

## Final notes

Besides the AS name, another hint about the infrastructure behind a website is the DNS name server. If we look at the full `dig` output for this site:

```console
$ dig blog.jiayu.co

; <<>> DiG 9.10.6 <<>> +tcp blog.jiayu.co
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63663
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 8, ADDITIONAL: 13

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1280
;; QUESTION SECTION:
;blog.jiayu.co.			IN	A

;; ANSWER SECTION:
blog.jiayu.co.		215	IN	CNAME	zealous-perlman-cb8cb5.netlify.com.
zealous-perlman-cb8cb5.netlify.com. 20 IN A	178.128.123.58

;; AUTHORITY SECTION:
netlify.com.		93020	IN	NS	dns2.p04.nsone.net.
netlify.com.		93020	IN	NS	dns4.p04.nsone.net.
netlify.com.		93020	IN	NS	ns02.netlifydns.com.
netlify.com.		93020	IN	NS	dns3.p04.nsone.net.
netlify.com.		93020	IN	NS	dns1.p04.nsone.net.
netlify.com.		93020	IN	NS	ns04.netlifydns.com.
netlify.com.		93020	IN	NS	ns01.netlifydns.com.
netlify.com.		93020	IN	NS	ns03.netlifydns.com.

;; ADDITIONAL SECTION:
dns1.p04.nsone.net.	8215	IN	A	198.51.44.4
dns2.p04.nsone.net.	14025	IN	A	198.51.45.4
dns3.p04.nsone.net.	14025	IN	A	198.51.44.68
dns4.p04.nsone.net.	8215	IN	A	198.51.45.68
ns01.netlifydns.com.	622	IN	A	45.54.30.1
ns01.netlifydns.com.	622	IN	AAAA	2607:f740:e630::1
ns02.netlifydns.com.	622	IN	A	45.54.30.65
ns02.netlifydns.com.	622	IN	AAAA	2607:f740:e630:4::1
ns03.netlifydns.com.	622	IN	A	45.54.30.129
ns03.netlifydns.com.	622	IN	AAAA	2607:f740:e630:8::1
ns04.netlifydns.com.	622	IN	A	45.54.30.193
ns04.netlifydns.com.	622	IN	AAAA	2607:f740:e630:c::1

;; Query time: 373 msec
;; SERVER: 10.10.10.1#53(10.10.10.1)
;; WHEN: Tue Oct 02 12:37:08 IST 2018
;; MSG SIZE  rcvd: 522
```

We can see "netlify" all over the output, even though we couldn't tell from the AS name.

## Appendix

The code in this post as a gist:

{{< linkpreview title="Bash URL to ASN lookup" description="Bash URL to ASN lookup. GitHub Gist: instantly share code, notes, and snippets." url="https://gist.github.com/yi-jiayu/0fcc8fda6b6311a537e17539c7468441" >}}
