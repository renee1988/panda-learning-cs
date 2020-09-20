---
title: 'Asynchronous JavaScript'
date: 2020-09-20
permalink: /front-end/async-js
tags:
  - js
  - front end
  - async
collection: front_end
excerpt: "Asynchronous Programming with JavaScript"
---

## Parallel vs. Async

**Parallel vs Non-parallel: Roller coster example**

- You are waiting inline to ride the roller coster, you are the only one allowed to ride it, even though there are empty seats in the roller coster → Non-parallel
- You are waiting inline to ride the roller coster, you and other 20 people are allowed to ride it at the same time, you are experiencing the ride at the same time → parallel

In the computing sense, parallelism is expressed through **threads**

- You can have one thread running on one CPU core in your system
- They are like queues of actions/operations that needs to happen
- At any given instance, one core could be doing one of these operations and at the exact same moment, another operation can happen at a different core
- Because we don’t have infinite number of cores, the OS has a layer of virtual threads which take care of scheduling the events across the available cores as much in parallel as possible

Parallelism is about optimization → get things done faster

**Asynchronicity → single thread**

- The programs inside our JavaScript run entirely on a single thread, even though the browser (or Node engine) can access to multiple threads
- At any given instance, there is one-line of JavaScript running in the JavaScript engine
- You could run JavaScript in multiple instances (which might look like multi-threading), but those programs can’t communicate with each other (e.g., web worker)

**Concurrency: multiple higher-level tasks happening within the same timeframe**

Async programming is to manage the concurrency in our JavaScript program.

## Callback
```javascript
setTimeout(function() {
    console.log("callback!");
}, 1000);
```
There are two parts of the above code:
- The first half of the above code is setTimeout function call
- The second half of the above code is wrapped inside a callback which is deferred to a later moment. At that later moment, it is picked up and continue the program from where we left off.

**Example exercise:**
1. Request all 3 files at the same time,
2. Render them ASAP
3. Render them in proper order: file1, file2, file3
4. Output “Complete” after all 3 are done
```javascript
function fakeAjax(url, cb) {
    const fake_responses = {
        file1: 'first text',
        file2: 'second text',
        file3: 'third text',
    };
    const randomDelay = (Math.round(Math.random() * 1e4) % 8000) + 1000;
    console.log("Requesting: " + url);

    setTimeout(function(){
        cb(fake_responses[url]);
    },randomDelay);
}

function output(text) {
    console.log(text);
}

const responses = new Map();
// Ugly but gets the job done :)
function handleResponse(file, content) {
    if (!responses.has(file)) {
        responses.set(file, content);
    }
    const files = ['file1', 'file2', 'file3'];
    for (let i = 0; i < files.length; i++) {
        if (!responses.has(files[i])) {
            return;
        }
        if (typeof responses.get(files[i]) === 'string') {
            output(responses.get(files[i]));
            responses.set(files[i], false);
        }
    }
    output('Complete');
}

function getFile(file) {
    fakeAjax(file, function(text){
        handleResponse(file, text);
    });
}

// Request all files at once in "parallel"
getFile("file1");
getFile("file2");
getFile("file3");
```
## Thunks
From synchronous perspective, a thunk is a function that has everything already that it needs to do to give you some value back → a function with some closure state keeping track of some value(s) and giving you those values whenever you call it.
```javascript
// Synchronous thunk
function add(x, y) {
    return x + y;
}

const thunk = function() {
    return add(10, 15);
}

thunk();
```
Asynchronous thunk is a function that doesn’t need any parameters except a callback function to do its job.
```javascript
function addAsync(x, y, cb) {
    setTimeout(function () {
        cb(x + y);
    }, 1000);
}

const thunk = function (cb) {
    addAsync(10, 15, cb);
};

thunk(function(sum) {
    return sum;
});
```
**Example exercise:**
```javascript
// Active thunk
function getFile(file) {
    let text;
    let fn;
    fakeAjax(file, function (response) {
        if (fn) {
            fn(response);
        } else {
            text = response;
        }
    });
    return function (cb) {
        if (text) {
            cb(text);
        } else {
            fn = cb;
        }
    };
}

const thunk1 = getFile("file1");
const thunk2 = getFile("file2");
const thunk3 = getFile("file3");

// Request all files at once in "parallel"
thunk1(function (text1) {
    output(text1);
    thunk2(function (text2) {
        output(text2);
        thunk3(function (text3) {
            output(text3);
            output('Complete');
        });
    });
});
```
Thunk is using the closure to maintain a state of something, which eliminates time as a complex factor of state.

## Promises

Promise example: restaurant ordering. When you pay a meal at a fast food restaurant, you will get a receipt for the food you paid for. This receipt is the placeholder of your future food, it is a promise that the restaurant owes a meal.

**Promise → a placeholder to eliminate time as a concern wrapped around the future value.**
```javascript
function trackCheckout(info) {
    return new Promise(
        function(resolve, reject) {
            // attempt to track the checkout
            // ...
            // if successful, call resolve
            // otherwise, call reject
        }
    );
}
```
**Promise trust:**

- only resolved once
- either success or error
- messages passed / kept
- exceptions become errors
- immutable once resolved

**Promise flow control → promise chaining**
```javascript
doFirstThing                        doFirstThing()
    then doSecondThing                  .then(function() {
                                            return doSecondThing();
                                        })
    then doThirdThing                   .then(function() {
                                            return doThirdThing();
                                        })
    then complete                       .then(complete, error);
or error
```
**Example exercise:**
```javascript
function getFile(file) {
    return new Promise(function (resolve, reject) {
        fakeAjax(file, resolve);
    });
}

const promises = ['file1', 'file2', 'file3'].map(getFile);
promises
    .reduce((acc, curr) => {
        return acc.then(function () {
            return curr;
        }).then(output);
    }, Promise.resolve()
    .then(function () {
        output('Complete');
    });  
    
// Note: resolve function is NOT the output function, it is a function
// that fires off an asynchronous function under the covers that tells
// the built-in JS Promise to go through a list of "then" handlers
// registered and execute them.
```
**Abstractions**
- `Promise.all`
- `Promise.race`
```javascript
// Race example: implement a timeout
// Some async action returns a promise
function someAsyncAction() {...}
Promise.race([
    someAsyncAction(),
    new Promise(function executor(_, reject) {
        // 3 seconds timeout: if 3 seconds passes,
        // promise from someAsyncAction is not
        // resovled, reject.
        setTimeout(function () {
            reject('Timeout!');
        }, 3000);
    });
]);
```
## Generators
All normal functions in JavaScript has a “**run-to-completion**” semantic → one of the most important JavaScript characteristics is that it allows us to reason about our code in a single-threaded fashion, never need to worry about two functions one interrupting the other, corrupting the shared memories.
- A generator is a syntactic form of declaring a state machine.
- A generator can be thought of a **pause-able function**.
```javascript
function* gen() {
    console.log('Hello');
    yield; // pause
    console.log('World');
}

const it = gen();
it.next(); // Hello
it.next(); // World
```
**Messaging**
```javascript
function* main() {
    yield 1;
    yield 2;
    yield 3;
    return 4;
}
const it = main();
it.next(); // returns {value: 1, done: false}
it.next(); // returns {value: 2, done: false}
it.next(); // returns {value: 3, done: false}
it.next(); // returns {value: 4, done: true}
```

```javascript
function coroutine(g) {
    const it = g();
    return function() {
        return it.next.apply(it, arguments);
    };
}

const run = coroutine(function* () {
    const x = 1 + (yield);
    const y = 1 + (yield);
    yield (x + y);
});

run(); // generator is paused to wait for a value for const x = 1 + (yield);
run(10); // this call completes const x = 1 + (yield); -> 11
console.log(
    'Meaning of life: ' + run(30).value
); // Meaning of life: 42
```
**Asynchronous generators**
```javascript
function getData(d) {
    setTimeout(function () {
        run(d);
    }, 1000);
}

// Synchronous looking asynchronous code :)
const run = coroutine(function* () {
    const x = 1 + (yield getData(10));
    const y = 1 + (yield getData(30));
    const answer = yield getData(
        'Meaning of life: ' + (x + y)
    );
    // After ~2 seconds, print Meaning of life: 42
    console.log(answer);
});

run();
```
**Promise + Generator → yield promise**
```javascript
async function foo() {
    await ajax(...);
}
foo();
```
**Example exercise:**
```javascript
async function getFiles() {
    const p1 = getFile('file1');
    const p2 = getFile('file2');
    const p3 = getFile('file3');
    
    output(await p1)
    output(await p2)
    output(await p3)
    
    output('Complete');    
}
```
> Do I need anything to happen in parallel? If so, store those things into intermediate promises and then sequence out the response.

## Observables
Example: spreadsheet

An observable is similar to a chain of calculated fields in a spreadsheet. In a spreadsheet, you can have a data in a field and a calculated data in another field. → The calculation chain is a flow of data.

**An observable is an adapter hooked onto an event source that produces promise every time there is an event coming through.**

In our code, we can declare an observable as our data source (data stream), and we can subscribe to the observable in one or more locations in our system.

```javascript
// Example using RxJS
// Rx.Observable.fromEvent takes in a DOM element and
// a DOM event. It hooks the event name to the element
// Every time when the event fires, it pumps a piece of
// data through the observable.
const observable = Rx.Observable.fromEvent(btn, 'click');
observable
    .map(function mapper(evt) {
        return evt.target.className;
    })
    .filter(function filterer(className) {
        return /foobar/.test(className);
    })
    .distinctUntilChanged()
    .subscribe(function (data) {
        const className = data[1];
        console.log(className);
    });
```
[RxJS Examples Reference](rxmarbles.com) 
## Communicating Sequential Processes

CPS models concurrency with channels.

- Channel is similar to a stream / pipe without buffer size but with built-in back pressure
    - back pressure: The two ends of the stream cannot communicate with each other, the only way to tell one end to stop sending data is to block the end → back pressure
    - back pressure is a reversed way of communicating from consumer to producer to tell the producer to stop producing/pushing data.
- Channel can only take one message at a time → blocking channels
    - You can’t send me something until I am ready to take it.
    - I can’t take anything until you are ready to send it.

CSP models your application with lots of tiny independent processes.

- There are times when the processes need to coordinate with each other, send messages to each other. After the coordination is done, they are unblocked and go back to being independent again.
- In JavaScript world, things aren’t running independently as above, but generators is close to it.
    - Generator can block itself but not affect any other part of the application.
    - We can have a bunch of generators running in different parts of our application, they are all independent of each other. At some point in time, two generators want to coordinate with each other, they need a communication channel where they can both block each other, waiting for each other to show up. Once the message is shown up and sent to each other, they will be unblocked and go back to be independent again.

```javascript
const ch = chan();

function* process1() {
    yield put(ch, 'Hello');
    const msg = yield take(ch);
    console.log(msg);
}

function* process2() {
    const greeting = yield take(ch);
    yield put(ch, greeting + ' World');
    console.log('Done!');
}
// Hello World
// Done!
```

**Event channels**

```javascript
function fromEvent(element, eventType) {
    const ch = csp.chan();
    ${el}.bind(eventType, function (e) {
        csp.putAsync(ch, e);
    });
    return ch;
}

csp.go(function* () {
    const ch = fromEvent(el, 'mousemove');
    while (true) {
        const e = yield csp.take(ch);
        console.log(
            e.clientX + ', ' + e.clientY
        );
    }
});
```