---
title: 'HTTP vs. HTTPs'
date: 2020-09-20
permalink: /full_stack/http-https
tags:
  - http
  - https
  - full stack
excerpt: "Differences between HTTP and HTTPs"
collection: full_stack
---

# HTTP

HTTP is an application layer protocol, it is stateless.

HTTP consists of
- Start-line: HTTP method, URI, HTTP version / HTTP version, Status code
- HEADER: HOST, ACCEPT, ACCEPT-LANGUAGE
- BODY: the content transferred between client and server is not encrypted

# HTTPs

**HTTPs = HTTP over TLS/SSL**

## SSL certificate, public key encryption

1. browser asks the website to identify itself
1. web server sends a copy of the certificate to browser to authenticate the identity of a website
1. browser will check and make sure the website is trustworthy
1. SSL session is established and the data are encrypted during the transfer.