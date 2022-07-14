---
title: What I have learn on DNS
description: Things that I learned about DNS
author: John Jerald De Chavez
date: 2022-07-14T02:14:03.681Z
tags:
  - dns
---
[What is DNS](https://www.cloudflare.com/learning/dns/what-is-dns/)
The Domain Name System (DNS) is the phonebook of the Internet. Humans access information online through domain names, like nytimes.com or espn.com. Web browsers interact through Internet Protocol (IP) addresses. DNS translates domain names to IP addresses so browsers can load Internet resources.

Common issue when working on DNS\
On Chrome:

> This site can't be reached \
> **cp-upload-staging.digiteer.dev**’s server IP address could not be found.\
> **ERR_NAME_NOT_RESOLVED**

On FireFox:

> Hmm. We’re having trouble finding that site.\
> We can’t connect to the server at cp-upload-staging.digiteer.dev.

To verify if the hostname is unreachable we can use tool like:\
[Dig](https://digwebinterface.com/) - Online DNS lookup tool

On Dig website, enter the hostname or ip address

`cp-upload-staging.digiteer.dev`

Set type as [CNAME](https://www.cloudflare.com/learning/dns/dns-records/dns-cname-record/) then click Dig button, and you will got result of:

> **[cp-upload-staging.digiteer.dev@8.8.4.4 (Default):](<>)**\
> cp-upload-staging.digiteer.dev.	300 IN	CNAME	d2iuzkbz6gz75k.cloudfront.net.

When there is result, means the hostname is ok, probably on your AWS infrastructure