---
title: 'This Reference in JavaScript'
date: 2020-09-20
permalink: /front-end/this-js
tags:
  - js
  - front end
  - this
collection: front_end
excerpt: ""
---

## This in Global Environment

- Under "use strict", 'this' is undefined.
- Otherwise, 'this' refers to window. 

**Example exercise:**

```javascript
'use strict'
function foo() {
        console.log(this);
}

foo(); // undefined
```

## This in Object

- If 'this' is directly called by the upper level object, 'this' refers to the upper level object.
- Otherwise, 'this' refers to global environment (window).

**Example exercise:**

```javascript
const foo = {
        name: 'Jack',
        fn: function() {
            console.log(this);
            console.log(this.name);   
        },
        classmate:{
            name: 'Annie',
            fn: function(){
                console.log(this.name);
            }
        }
}

var fn1 = foo.fn; 
fn1(); //window, undefined

foo.fn(); //{name: 'Jack', fn: f}, 'Jack'

foo.classmate.fn(); //'Annie'
```

- What's more, if we call the function of one object in another object, the conclusion of 'this' in object still hold.

**Example exercise:**
```javascript
const student1 = {
    name: 'Jack',
    fn: function() {
        return this.name 
    }
}

const student2 = {
    name: 'Annie',
    fn: function() {
        return student1.fn() 
    }
}

console.log(student1.fn()); // 'Jack'
console.log(student2.fn()); // 'Jack'
```

## Explicitly Bind (Call, Apply, Bind and New)

- For Call and Apply, they change the reference of 'this' and execute the function.
- For Bind, it changes the reference of 'this' but not executes the function.
- For New, when we new a object, actually we refer 'this' of construtor function to our instance (protoType and \_\_proto\_\_).

**Example exercise:**
```javascript
const student1 = {
    name: 'Jack',
    fn: function() {
        return this.name 
    }
}

const student2 = {
    name: 'Annie', 
}

console.log(student1.fn.bind(student2)); //fn
console.log(student1.fn.apply(student2)); //Annie
console.log(student1.fn.call(student2)); //Annie
```

- When we do explicitly bind again, the reference will update.

**Example exercise:**
```javascript
function Student(name){
    this.name = name
}

const student1 = {}
let getName = Student.bind(student1)
getName('Jack');
console.log(student1.name); //'Jack'
let student2 = new getName('Annie');
console.log(student2.name); //'Annie'
```

## Arrow Function

- In arrow function, 'this' refers to the outer function or environment.

**Example exercise:**

```javascript
const student = {
    name: 'Jack',
    fn: () => {
        console.log(this.name);
    },
    fn2: function() {
        return ()=>{
            console.log(this.name);
        }
    }
}

student.fn(); //undefined
student.fn2()(); //Jack
```

