---
title: Httpie
description: HTTPie (pronounced aitch-tee-tee-pie) is a command-line HTTP
  client. Its goal is to make CLI interaction with web services as
  human-friendly as possible. HTTPie is designed for testing, debugging, and
  generally interacting with APIs & HTTP servers.
author: John Jerald E. De Chavez
date: 2022-05-07T07:24:00.000Z
tags:
  - http
  - cli
  - resource
  - post
installations:
  - title: test
    command: test
---
Httpie a simple yet powerful command-line HTTP and API testing client for the API era.
It used for testing and debugging of APIs, HTTP servers, and web services.

### Getting Started

Arch

`pacman -Syu httpie`

Python (Please make sure have Python 3.7 or newer)

`python -m pip install --upgrade pip wheel`

`python -m pip install httpie`

Homebrew\
`brew update`

`brew install httpie`

For more information, check <https://httpie.io/docs/cli/installation>

### Usage

Hello World:

```
https httpie.io/hello

Result:
`
HTTP/1.1 200 OK
CF-Cache-Status: DYNAMIC
CF-RAY: 691e8b1169266c33-SIN
Connection: keep-alive
Content-Encoding: gzip
Content-Type: application/json; charset=utf-8
Date: Mon, 20 Sep 2021 22:38:20 GMT
Expect-CT: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
NEL: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
Report-To: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v3?s=9Q04fRZBtyXnjxCOM2Nlaxyj%2BbpAsrk7Wc%2BSd%2Bb28A3JORiAEEd%2BH6MRVURGy%2BPK0Ve4nLTvCgNBgTkXIeGzoU5602Zw3Jywo67KRUctWbKIOssZ9GBkrl7joio%3D"}],"group":"cf-nel","max_age":604800}
Server: cloudflare
Transfer-Encoding: chunked
age: 0
alt-svc: h3=":443"; ma=86400, h3-29=":443"; ma=86400, h3-28=":443"; ma=86400, h3-27=":443"; ma=86400
cache-control: public, max-age=0, must-revalidate
etag: W/"105-tuZg7HTShsOZ9QXSZS0pqWYQ2Ac"
x-matched-path: /api/hello
x-vercel-cache: MISS
x-vercel-id: sin1::iad1::2v7jj-1632177497839-4b362b7c99e8

{
    "ahoy": [
        "Hello, World! 👋 Thank you for trying out HTTPie 🥳",
        "We hope this will become a friendship."
    ],
    "links": {
        "discord": "https://httpie.io/chat",
        "github": "https://github.com/httpie",
        "homepage": "https://httpie.io",
        "twitter": "https://twitter.com/httpie"
    }
}
`
```

\
Add Headers specially Authorization:

`http http://localhost:8062/admin/lesson-roadmap 'Authorization:Bearer [Token]'`

Custom HTTP method, HTTP headers and JSON data:

`http PUT pie.dev/put X-API-Token:123 name=John`

Submitting forms:

`http -f POST pie.dev/post hello=World`

Upload a file using redirected input:

`http pie.dev/post < files/data.json`

Download a file and save it via redirected output:

`http pie.dev/image/png > image.png`