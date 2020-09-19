---
title: 'HTTP2'
date: 2020-09-20
permalink: /full_stack/http2
tags:
  - http2
  - https
  - full stack
excerpt: "HTTP2 overview"
collection: full_stack
---
## HTTP/1.x drawbacks

- Clients need to use multiple TCP connections to achieve concurrency and reduce latency.
- Request and response headers are not compressed.
- There is no effective resource prioritization.

## HTTP/2.0

- Reduce latency by enabling full request and response multiplexing
- Minimize protocol overhead via efficient compression of HTTP header fields
- Add support for request prioritization and server push

### Terminology

- Stream: a bidirectional flow of bytes within an established connection.
- Message: a complete sequence of frames that map to a logical request or response message.
- Frame: smallest unit of communication in HTTP/2.0
    - contains a frame header that indicates to which stream it belongs.
- All communication is performed over a single TCP connection that can carry any number of streams.
- Each stream has a unique id and optional priority information that is used to carry bidirectional messages.
- Each message is a logical HTTP message, a request or response, which consists of one or more frames.
- Frames from different streams may be interleaved and then reassembled via the embedded stream id in the header of each frame.

### Request and response multiplexing

With HTTP/1.x:
- if the client wants to make multiple parallel requests to improve performance, multiple TCP connections must be used.
- only one response cane be delivered at a time (response queuing) per connection.

With HTTP/2.0:
- request and response multiplexing allows client and server to break down an HTTP message into independent frames, interleave them, and then reassemble them on the other end.

### Stream prioritization

- Each stream may be assigned an integer weight between 1 and 256
- Each stream may be given an explicit dependency on another stream

### One connection per origin

- Most HTTP transfers are short and bursty, whereas TCP connection is optimized for long-lived, bulk data transfers.
- Reusing the same connection reduces the overall protocol overhead and memory and processing footprint along the full connection path (i.e., client --- ... --- server)

### Flow control

- prevents the sender from overwhelming the receiver with data that may not want / be able to process
- client may have requested a large video stream with high priority but the user pauses the video and the client now wants to pause or throttle its delivery from server.
- a proxy server may have a fast downstream and slow upstream connections, wants to regulate how quickly the downstream delivers data to match the speed of upstream to control its resource usage.

### Server push

- Server can send multiple responses for a single request.
  - inlined CSS, Javascript or any other assets via a data URI:
```html
<img src="data:image/gif;blahblahblah" />
```
- Pushed resources must obey the same-origin policy

### Header compression

- HTTP/2.0 compresses request and response header metadata using HPACK compression format:
    - header fields are encoded via a static Huffman code.
    - both client and server maintain and update an indexed list of previously seen header fields, establishes a shared compression context.
- static table provides a list of common HTTP header fields.
- dynamic table is initially empty and is updated based on exchanged values within a particular connection.

## References

- [HTTP1. vs HTTP2 Demo](https://http2.akamai.com/demo)
- [HTTP2 explained](https://http2.akamai.com/)
