---
title: 'Thinking reactively'
date: 2020-09-20
permalink: /rx_js/thinking-reactively
tags:
  - js
  - async
  - rxjs
  - front end
excerpt: "Thinking reactively to deal with the composition of asynchronous data flows."
collection: rx_js
---
## Synchronous vs. Asynchronous computing
The main difference between synchronous and asynchronous computing/code is **latency/waitime**.

### Blocking code
Synchronous execution occurs when each block of code must wait for the previous block to complete before running.
- Easy to implement
- Easy to understand
- Easy to debug

JavaScript is a single-threaded language. Writing blocking code creates awful user experience.
- waiting for AJAX call to return
- waiting for database operations to complete
The entire application would pause/sit idle waiting for the data to be loaded and wasting precious computing cycles that could be executing other code.

Other than horrible user experience, browsers may deem your scripts unresponsive after a certain period of inactivity and terminate them.

### Non-blocking code with callback functions
<img src="/images/rx-js-cb.png">

As a single-threaded language, JavaScript provides callback functions to tackle the problem of blocking for long-running operations to complete by allowing you to provide a handler function that the JavaScript runtime will invoke once the data is ready to use.
- JavaScript callback functions create **inversion of control** where functions call the application back insetad of the other way around.
- Inversion of control refers to the way in which certain parts of your code receive the flow of control back from teh runtime system.

Callback functions allow you to invoke code asynchronously, so that the application can return control to you later. This allows the program to continue with any other task in the meantime.

### Time and space
- Synchronous functions allows us to reason directly about the state of the application
- Asynchronous code forces us to reason about its **future** state

For example, if you have three functions performing three independent tasks, then executing them in any order wouldn't matter. However, if they are sharing some global state, their behavior would be determined by the order in which they were called -> ***Side Effect***

### Callback or RxJS?
- If your script issues a single remote HTTP request, RxJS is an overkill, callbacks remain the perfert solution.
- RxJS begins to shine when implementing state machines of advanced complexity such as:
  - dynamic UIs: rich UI made up of several widgets on the page that interact with each other
  - service orchestration: orchestrate the execution of several business process that consumes several microservice, data mashups

```javascript
// Example: callback hell
ajax(
    '<host1>/items',
    items => {
        items.forEach(item => {
            ajax(`<host2>/items/${items.getId()}/info`, dataInfo => {
                ajax(`<host3>/files/${dataInfo.files}`, processFiles)
            });
        });
    },
);
```

## **TO BE CONTINUED**
