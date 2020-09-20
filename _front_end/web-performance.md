---
title: 'Web Performance'
date: 2020-09-20
permalink: /front-end/web-performance
tags:
  - js
  - front end
  - performance
collection: front_end
excerpt: "Learn how to boost your web application performance"
---
User Experience: first impression matters, a lot.

## The Big 14
1. **Fewer HTTP requests during page load**
2. **Use CDN**: If you have static resources that do not require the server to figure out what its content is, you can store a versioned file in CDN.
    1. CDN has more points of presence than you do (you might have your web server in a couple of data centers, most CDN’s have multiple locations throughout world)
    2. Shared caching: if I require a JS library from a google location, and a thousand of other sites the user visits also reference the same JS library. In general, I will have this library more likely in my browser cache.
3. **Expires/Cache-Control header**: You want to make sure that you are sending out requests with proper headers that correspond to your intent for a piece of content.
4. **Gzip**
5. **Stylesheets at Top**: Browser will block rendering of content until it knows about all the stylesheets so that it doesn’t paint things incorrectly and has to redo that work.
6. **Scripts at Bottom**: Rearranging the order of the markdowns in HTML is a priority indication to the browser of which resources are most important to you.
7. **~~CSS EXpressions~~**
8. **Externalize JS/CSS: caching**
9. **Fewer DNS Lookups**
10. **Minify JS/CSS**
11. **~~Redirects~~**
12. **~~Duplicate Scripts~~**
13. **ETags**: fingerprint for a particular resource → conditional loading: if I have a fingerprint for a file, I can ask server that here is the fingerprint that I have for the server, is the current fingerprint of the file different? If it is, send me the file, otherwise it tells me the file hasn’t changed.
14. **Cacheable Ajax**: if you are requesting resources, you can add on an additional layer of expiration duration.

### Images

Image optimization: e.g., png: lossless compression [Image sprites](https://www.w3schools.com/css/css_image_sprites.asp):
- a collection of images put into a single image.
- A web page with many images can take a long time to load and generates multiple server requests.
- Using image sprites will reduce the number of server requests and save bandwidth.

Inline images:
```css
#foo {
    background: url(data:image/png;base64, XXXXXXXX);
}
```
- When you inline an image, you do reduce the amount of external requests happening.
- However you also tie the cache-ability of that image to your CSS file. In some cases, your CSS file changes way more frequently than your image file, then you will cause more extra downloads.

## Middle-End: Architecture & Communication
<img src="/images/web-performance-1.png">

- Backend can be anything, Java, .Net, NodeJS, GoLang.
- Middle-end includes
    - Templating
    - URL routing
    - Data validation
    - Data formatting
    - Headers
    - Caching
- Think about backend as a blackbox
    - it has APIs
    - there is a state machine inside of it, manages states through sessions, manages states saving values to the database, etc.
    - the architecture of the backend doesn’t matter
    - what is exposed on top of the backend is a headless stateful API with response of JSON data
    - you have a contract between the backend and the middle-end, what type of data is exchanged, etc.
        - Middle-end: Hi backend, I know we are in state ABC, the user now requests state DEF, please handle the request.
        - Backend: handle and decide whether it is a valid state change, extracting the state management logic away.

### Architecture
Single-page application:
- Load markup or the shell of your application once
    - Serve up an initial view
    - Once we have a fully initialized application inside the browser, we don’t need to re-roundtrip to the browser to ask it to re-create the whole new page for us (for example, Facebook comment feature)
        - The markup to create a comment is simple and straightforward, but when you have a list of these comments, you have a list of the repetitions of the same markup.
        - You have a choice of how to architect things, either make a roundtrip to the server and ask for the whole comment list or worse the whole page to be re-rendered, or you can choose to have the data to go into the markup and the markup → simple templating task to regenerate the items to update to the DOM.
            - **You can use JavaScript templating engines ([Mustache](https://mustache.github.io/), [Handlebars](http://handlebarsjs.com/), etc.)**
            - **You can use data-binding**

Data validation:

There are often two copies of the **stateless** data validation logics exist in your application

- one in the backend
- one in the frontend
- there will be times when they are not sync’ed, one is outdated.

Middle-end is the place where you can use server-side javascript and put the data validation logic.

If the validation is stateful, e.g., is this a unique email address, it can’t be done in the middle-end, it has to be done in the backend, because your database doesn’t live on your client side, it requires a roundtrip to your server.

### JSON, Ajax & Web Sockets

Ajax is a full request-response cycle at the HTTP layer, it opens up a new connection on the server and therefore it is taking extra resources, extra HTTP packets over the wire, extra connection resources in your browser and in your server.

Web socket creates one request-response cycle, as soon as it has a good established connection/handshake between client and server:
1. the client that decides to have a socket connection sends the request over the  HTTP socket **open**
2. it tells the server it wants to switch to web socket protocol
3. the server agrees on the versions and does security checks/handshakes
4. now you have a persistent socket between the client and server, which doesn’t require any HTTP overhead bytes (e.g., headers) to communicate.

Web socket gives us two-way communication which Ajax couldn’t offer (you have to do long polling): emails, push notifications, chats, gaming, etc.

- When something happens on the server, we need to notify everybody that is listening, we need to push out that piece of information to the client, we don’t want to wait for the client to ask for it.
- Socket communication is often referred to as a “real-time” communication, it has much lower latency as oppose to an Ajax request.

## Front-end: Resource Loading

Resources:
- JavaScripts files
- CSS files
- Static images

### Preloading

Preloading is a technique that takes advantage of browser cache/memory, we know we are going to need these resources later, we can just ask the web server to give you these resources now, we will cache these resources.

- When end-users get later to these resources, they are already there, can be loaded super fast.
- However your initial page load time will be slower. → Lazy preloading

```html
# Hint for preloading to the browser when it is
# parsing this markup.
<link rel="prefetch" href="something.png">
```

Extra notes:
- there is no event/way for you to know when the resource is prefetched in your JavaScript.
- DNS prefetching in Chrome: Chrome will automatically perform DNS pre-caching for looking ahead in the markup, see if there are any unrecognized domain name or host name, and automatically goes and fetch them.
    - If you have hrefs in your markup, it will look at the domain name and do a DNS pre-caching if necessary so that later on when your users click that link, they can get to that link quicker.

### Lazy Loading

Lazy loading can also be referred as on-demand loading, postloading, it waits until we are sure that someone needs these resources before downloading.
- For example, calendar widget, until the user clicks the calendar widget component, you don’t load any of the code that handles calendar widget rendering.

### Parallel Loading

When you get many scripts that need to be loaded, all the scripts that you load on initial page load need to be loaded dynamically.

Parallel loading is essentially about preserving execution order.
- By browser’s default behavior, if you ask for two scripts to load, the browser will download both of them in parallel, execute the first one that is finished downloading.
- Browser doesn’t preserve execution order

```javascript
function allScriptsLoaded() {...}

function loadScript(source, done) {
    const src = document.createElement('script');
    src.src = source;
    src.async = false; // by default it is true
    document.head.appendChild(src);
    if (done) {
        src.onload = done;
        src.onreadystatechange = function () {
            if (src.readyState === 'loaded' || src.readyState === 'complete') {
                done();
            }
        }
    }
}

loadScript('react.js');
loadScript('some-widget1.js');
loadScript('some-widget2.js', allScriptLoaded);
```
Note in the above code snippet, if `some-widget1.js` fails to load, then `some-widget2.js` might be loaded successfully but it will never be executed.

**Why Dynamic Loading?**

**document.ready is way faster than put your scripts in the HTML markup.**
- Browser has to assume that the script element in the markup might contain a **`document.write`**
- If there is a `document.write` in your script, browser doesn’t know but you might inject a whole new markup into the document, which changes entirely how browser interprets your page → Therefore it can’t fire the document.ready event until it finishes loading the scripts when it sees something in the markup.

## Abstractions

### Object-orientation is slow(er)
Object-oriented code in the context of running JavaScript in browser is not working the same way as the object-oriented code written in a compiled language.
- It is a live interpreted link of inheritance rather than a compiled link of inheritance.
- In OOP world, you tend to create more abstraction than necessary (hoping that someday it will be necessary). You end up making more function calls, more abstraction layers which tend to slow things down.

## Animation

### JS to CSS
Most of the time if you can perform animation in CSS, you move to CSS. CSS engine is more reliable than your JavaScript.
`setTimeout` & `setInterval` cannot enforce the exact time the callback is executed, the browser tries to put the callback in the event loop at the required time, but it is not guaranteed.

### requestAnimationFrame
What `requestAnimationFrame` does is that I ask the browser that I get some code that I want to run in very next time the browser is about ready to paint the update to the screen (browser paints the updates ~60 times per second).
- browser controls the frame rate
- much more reliable than setTimeout / setInterval
- browser will be able to pause the loop when the tabs in the background are closed or not visible

## UI Thread
Threaded programming is to have the CPU manage the different chains of operations and executions, they can go parallel and CPU can very quickly switch back and forth, or even it can have multi cores, process multiple operations at the same time.

JavaScript forever will be SINGLE-THREADED.

### Async vs. Parallel
JavaScript has asynchronous code that happens in an event loop, but **it doesn’t have parallelism**. It is not multi-threaded. You can spin up two NodeJS instances that run on different processes, but within a single execution context of JavaScript, it is single-threaded.

The way JavaScript handles the asynchronous code is in an event loop:
- it does single-threaded execution
- it has different operations in different queues that it might choose to do
- it will only do it one at a time
- For example, NodeJs, you can think of it as a big pool of connections asking for things, and NodeJs has an ordered way to handle these requests/tasks. It can switch quickly between different queued tasks.

UI Thread is single-threaded, there are several different systems inside the browser that are all sharing the same thread, meaning they can have only one of the tasks from these systems at a time.
- CSS rendering engine
- JS engine
- DOM
- Garbage collector

For example, if the JS engine is executing some task at a given moment, but CSS rendering engine comes in and wants to repaint a frame at the same moment, CSS rendering engine has to wait for the JS engine to finish its ongoing task.

### “Threaded” JavaScript - Web worker
Web worker allows you to point to a particular JavaScript file and run it in a different thread. Whatever goes on in that thread won’t affect your current thread.
- it allows you to take long running JavaScript code (e.g., heavy calculation, data processing) and run it in a different thread at the same time as you run a CSS animation
    - You have all the cool CSS animations going on and at the same time you have huge amount of communications going on with the server, JavaScript needs to handle different event handlers, etc.
    - You want a nice smooth CSS animation with all the server communication going on at the same time.
    - **You can put all your socket communications into a web worker which runs its own thread, it runs very fast.**
- the downside of using web workers is the communication layer between your current thread and the web worker may become your bottleneck if you have tons of different messages to communicate.

```javascript
// Pass a reference of your heavy-data-processing javascript file
// to the constructor of a web worker to instantiate it.
// JavaScript engine takes care of the followings:
// 1. spwan a new thread.
// 2. set up a new asynchronous communication channel in between
//    the main thread and the web worker thread.
var longThread = new Worker('heavy-data-processing.js');

longThread.onmessage = function (e) {
    var answer = e.data;
    // ... do stuff with the answer
};

longThread.postMessage({
    question: '...',
});

// heavy-data-processing.js
funcion calculate(question) {
    // ...
    return answer;
}

// self is the reference to the web worker
self.onmessage(function(e) {
    var question = e.data.question;
    var answer = calculate(question);
    self.postMessage(answer);
});
```

> Question: What is the difference between making an asynchronous request from the web worker and the main thread?

**Answer**: Inside the main thread, when you make the request to the browser, once it leaves the JS engine, it enters into the world with threaded handling where requests can be handled in parallel. However, as soon as that communication needs to come back to the JS engine, **it has to wait for the JS engine to have a free cycle**. The JS engine won’t have a free cycle until the UI thread is free. While if the request comes back to JS engine but there is a CSS animation going on in the UI thread, your request won’t be handled until the CSS engine finishes its ongoing task.

> Followup: In this case, it sounds like we should just use web workers always.

**Answer**: the communication channel between the main thread and the web worker is **STRING** based, and it is **COPY-ONLY**. Therefore if you have a lot of data to send, you end up with two copies of data in memory until the garbage collector comes along clear them out.

### Dynamic Memory Allocation - Garbage collector
JavaScript allows you to create variables, elements in the DOM, and it allows you to not delete those variables but simply to stop referencing them.

```javascript
var obj = {
    // ... lots of data, e.g., 1MB of data
};

// ... later

obj = null; // <- here we are setting the reference to null, not the object itself.

// What happens to that 1MB chunk of data?
// Someone has to come along and free up the memory.
```

Garbage collection is one of the tasks run in the UI thread. The more often the browser thinks the garbage collector needs to run, the more your code will be affected negatively performance-wise.

## References
- HTTP Archive: [https://httparchive.org/reports/state-of-the-web](https://httparchive.org/reports/state-of-the-web)
- Image sprite tool: [https://spritegen.website-performance.org/](https://spritegen.website-performance.org/)
- [https://robertnyman.com/2008/05/09/improve-your-web-site-performance-tips-tricks-to-get-a-good-yslow-rating/](https://robertnyman.com/2008/05/09/improve-your-web-site-performance-tips-tricks-to-get-a-good-yslow-rating/)
- [https://robertnyman.com/2010/01/15/how-to-reduce-the-number-of-http-requests/](https://robertnyman.com/2010/01/15/how-to-reduce-the-number-of-http-requests/)
- [https://jsperf.com](https://jsperf.com/)