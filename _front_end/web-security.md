---
title: 'Web Security'
date: 2020-09-20
permalink: /front-end/web-security
tags:
  - js
  - front end
  - security
collection: front_end
excerpt: "Learn how to attack and defend your web applications"
---

## Cross-site scripting

Injection attack: put content where that is designed for text and trick a system to read it as code and execute it.

It allows attackers to read data, or perform operations on users’ behalf.

Depending on what users are compromised

e.g., administrator → full system control → create administrator accounts → access database

### TYPES OF XSS

- Stored XSS: Code that executes attacker’s script is persisted
- Reflected XSS: Transient response from server causes script to execute (i.e., a validation error, notifications)
- DOM Based XSS: No server involvement is required (i.e., pass code in via queryParams)
- Blind XSS: Exploits vulnerability in another app (i.e., internal app, more vulnerable, less protection, can absorb public data that is melicious, log-reader), that attacher can’t see or access under normal means

### LOCATIONS FOR XSS ATTACKS

- User-generated rich text
- Embedded content (you can drop an iframe, or an object, flash, etc.)
- Anywhere user has control over a URL
- Anywhere user input is reflected back (i.e., “couldn’t find ___”)
- Query parameters rendered into DOM
- element.innerHTML = ___

### XSS DEFENSE

- Never trust user data:
    - directly in a script
    - in an HTML comment
    - in an attribute name
    - in a tag name
    - directly in a `<style>` block
- Sanitize the data before it is persisted (encoding)
    - React: `dangerouslySetInnerHTML` props field
- Use values as values not as code: careful with JS templating
- Content security policy (CSP)
    - Browsers can’t tell the difference between scripts downloaded from your origin vs. another. It is a single execution context/environment. (e.g., ReactJS can be from CDN, your own script can be from somewhere else)
    - CSP allows us to tell modern browsers which sources they should trust and for what types of resources
    - This information comes via a HTTP response header or meta tag
        - name: where you are allowed to get scripts
        - ‘self’: the origin that the frame has been served on
        - source: protocol + host + port
    - Useful CSP directives
        - child-src: child execution contexts (only allowed to spin up frames, workers from the following domains)
        - connect-src: what you can connect to (fetch, WebSocket, EventSource(HTTP long-polling connection))
        - form-action: a list of origins that you can `<form>` submit/post to
        - img-src, media-src, object-src: a list of origins you can get image, media, flash from
        - style-src: a list of origins external stylesheets can come from (bootstrap, etc.)
        - upgrad-insecure-requests: upgrades from HTTP to HTTPS
        - default-src: fallback, for when specific directive isn’t provided

```
Content-Security-Policy: script-src 'self' https://some.site.i.trust
                         [   name  ]       [         sources       ]
```

- CSP and ‘unsafe-inline’
    - Script tags embedded in HTML is the most common form of XSS.
    - Cryptographic nonces must be generated per page load and must change unpredictably.
- [helmetJS](https://helmetjs.github.io/):

    a collection of 12 smaller middleware functions that set HTTP response headers.

### MALICIOUS ATTACHMENTS

- PDF attachments: can modify memories
- Embedded malware in image: malware config file

## Cross-site request forgery

It takes advantage of cookies (basic authentication credentials) are passed along with requests.

```html
<img src="http://example.com/transfers/perform?accountFrom=12&accountTo=21" />
```

### COOKIE FLOW
```
Web application -------------------AUTHENTICATION (login)------------------> Web server

                        |----------AUTHENTICATION (cookie)-------|
```

Consider you have some web application, upon logging in, you get some cookies which will be stored in your web browser. Whenever you are making a request to the web server,

**the cookies are sent along with the requests automatically**

so that the server can verify your credentials.

### CROSS DOMAIN ACCESS CONTROLS

For example, in HTML, we have iFrames which allows you embed one website inside another. However the cross-iframe communications are not possible due to the

**Same Origin Policy**

. ← It is a security feature to make sure that an attacker website can’t arbitrarily make requests to other websites and access data across domains.

### CROSS-SITE REQUEST FORGERY ATTACKS

One domain is forging request to another in order to modify data.

For example, some user logs into his account on some vulnerable website (cookles like session id are saved on your browser), then visits another website and click some malicious link, his account gets altered.

```html
<form action="http://vulnerable.com/delete_account" method="POST" id="csrf-form">
  <input type="hidden" name="delete" value="1" />
</form>
<script>document.getEelementById('csrf-form').submit();</script>  
```

### CROSS-SITE REQUEST FORGERY PROTECTION → CSRF TOKENS

Anti CSRF tokens are generated randomly by web application backend and sent to the web application frontend.

- Proves you are authenticated
- Proves you are sending a request from an appropriate place

### CROSS-SITE REQUEST FORGERY PROTECTION → REQUEST ORIGIN

- Modern browsers send an Origin header which cannot be altered by client-side code with each request
- In cases where there is no Origin header, there’s almost always a Referer header
- When behind a proxy, you can usually get some information from Host and X-Forwarded-Host headers

### CROSS-SITE REQUEST FORGERY PROTECTION → CROSS-ORIGIN RESOURCE SHARING

- It is a bypass of Same Origin Policy
- It permits browser to send request from one domain to another

## Clickjacking

- A “UI redress attack”: some invisible button sitting on top of the actual button, and trick the user to perform a click
- Can be used to capture keystrokes as well

### CLICKJACKING PROTECTION

- X-Frame-Options: HTTP response header that informs the browser when it downloads the document that you are not allowed to put this inside another page:
    - `X-Frame-Options: DENY`
    - `X-Frame-Options: SAMEORIGIN`
    - `X-Frame-Options: ALLOW-FROM https://some-domain-i-trust.com`
- Chrome/Safari don’t respect `ALLOW-FROM`. Use `frame-ancestors` CSP directive instead

## Man-in-the-middle

For example, a user opens a browser in a public WiFi network (e..g., Starbucks free WiFi connection). The HTTP request goes from his device over the WiFi radio into the router and goes into the server.

Public WiFi: trusted forever by default

WiFi devices broadcast what they are looking for

Router as DNS: when you log onto that network, you trust that router as the first DNS server inline (translates host names into IP addresses) → All of your host names are now routed to the Starbucks router.

Consider an attacker comes into the Starbucks, connects to the public WiFi and scans through all the networks that everyone in the Starbucks is asking for and has been before. He can pop up a network name (Free XXX WiFi) and send garbage packets to the connection between target and the router, tries to de-authenticate the user from the router. The user starts to look for other networks they have seen before and eventually join the Free XXX WiFi, the attacker then becomes the man in the middle of all the requests the user sends out.

- Attacker can eavesdrop on and tamper with communication between you and one or more servers
- XSS at will
- Capture your credentials: try your credentials on other sites

### MAN-IN-THE-MIDDLE PROTECTION

→ HTTPS, encrypt data in flight

- A secret key in order to read or alter a request/response
- Certificates identify domains and require “Domain Validation”
- “Enhanced Validation” often requires government ID

## HTTPS

### CRYPTOGRAPHY

Two types of encryption involved: Symmetric encryption and Public Key encryption

- Symmetric encryption is very fast, has no practical limit on size of content.
- Generate encryption key on a per-connection basis
- How do we share this secret if we randomize the key? → Public key encryption
- Public key encryption: Private key & Public key
    - the public key is used to create encrypted messages
    - the private key is used to read encrypted messages
    - the server makes its public key available to anyone who wants to use it
        - it sends its public key and certificate to the client
        - the client and server compare “cipher suites”
        - the client generates a **session key** (symmetric encryption, private key) and uses the public key from the server to encrypt it and send it out with the requests
            - Session key is what’s used for encrypted data exchange

### TLS HANDSHAKE

> Client: Hi, I’d like communicate securely

> Server: Here is my certificate to prove my identity and my public key for you to use to encrypt the data sent to me

> Client: Your certificate checks out, it must really be you! Here is a big random number X...X encrypted with your public key, is my session key!

> Server: Great! I will be encrypting everything I say with X...X from now on!

→ OpenSSL is an industry standard library for crypto (algorithms, handshake, protocol, etc.)

- Generate a private key
- Generate a public key from private key
- Make a new certificate singing request
- Sign the certificate with your private key