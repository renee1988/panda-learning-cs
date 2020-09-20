---
title: 'Event loop'
date: 2020-09-20
permalink: /front-end/event-loop
tags:
  - js
  - front end
  - event loop
collection: front_end
excerpt: "Quick recap on event loop concept"
---

- Event loop is a mechanism that handles executing multiple chunks of your program over time
- At each moment, it invokes the JavaScript engine
- JavaScript engine has no sense of time
- It is an on-demand execution environment for any snippet of javascript
- Event loop is an array that acts as a queue
- Event loop breaks its works into tasks and executes them in serial
- Event loop does not allow parallel access and changes to shared memory
- JavaScript never shares memory across threads
- JavaScript is single-threaded, but browser is not
- Process 1 and process 2 run concurrently task-level in parallel, but their individual events run sequentially on the event loop queue