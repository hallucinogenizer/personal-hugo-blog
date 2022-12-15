---
title: "Curried Functions in JavaScript"
summary: "Functions that are called like this: `sum(a)(b)(c)`"
date: 2022-10-08T11:35:18+05:00
draft: false
---

You may have seen arrow functions written like this:
```javascript
const sum = (a) => (b) => (c) => a + b + c
```
This article will explain how in some cases, the above syntax can help us write clean, concise, and functionally pure code.
At first glance, one can estimate that the `sum` function above computes the sum of three numbers `a`, `b`, `c`. 

What is interesting about it is that it is called like this:
```javascript
sum(1)(2)(3)
```
unlike your day-to-day javascript function:
```javascript
sum(1, 2, 3) //output: 6
```

You may have two questions in your mind right now:
1. **How** does this arrow function work?
2. **Why** would you want to write an arrow function like this?

Let's answer both. You'll need to understand the **How** before being able to understand the **Why**.
## How
Let's start off with a simple example:
```javascript
const sum = (a) => (b) => a + b

sum(1)(2) //output: 3
```

### Function Definition
There are two arrow functions here if you notice. The first function returns the second function.
```javascript
const sum = (a) => (b) => a + b
```
The first arrow function takes `(a)` as an argument and returns another function.
```javascript
const sum = (a) => function
```
The second arrow function is the one that the first arrow function returns:
```javascript 
(b) => a + b // is a function
```
This second arrow function takes `(b)` as an argument and returns the sum (which is most probably a number).

To further drive home this point, let's convert both of these arrow functions into regular functions, starting with the first arrow function.
```javascript
// 2 arrow functions
const sum = (a) => (b) => a + b

//change outer (1st) arrow function to regular function
function sum(a) {
    return (b) => a + b
}
```
Now the distinction between both functions should be clearer. Let's convert the inner arrow function to a regular function as well to make the distinction crystal clear:
{{<highlight javascript "hl_lines=3-5">}}
function sum(a) {
    return 
        function(b) {
            return a + b
        }
}
{{</highlight>}}

### Function Usage
Now that we somewhat understand the function definition, let's understand its use:
```javascript
sum(1)(2) //output: 3
```
Remember that we recently realized that there are two functions at play here. That is why we see two sets of `()` round brackets here as well, because both functions are being called. But how?

Let's call only the first function and see what happens.
```javascript
sum(1) //output: a function
```
Here we have called only the first function, i.e. the function that takes `(a)` and returns another (inner) function. Let's store that returned (inner) function in another variable.
```javascript
const sum2 = sum(1)
```
Now `sum2` contains our second function `(b) => a + b`. 

Here comes the most important thing to understand. When we called `sum(1)`, the `sum` function computed the following:
```javascript
function sum(a) {
    return (b) => a + b
}
```
Since the value of `a=1` was passed to it as argument, it goes as follows:
```javascript
function sum(1) {
    return (b) => 1 + b
}
```
This is the function that is returned from `sum(1)` and is stored in `sum2`.
```javascript
sum2 = (b) => 1 + b
```

Now let's call `sum2` as well:
```javascript
sum2(2) //output: 3
```
Why is the output `3`? Because sum2 was:
```javascript
sum2 = (b) => 1 + b
```
So when we gave it `b=2`, it did the following computation:
```javascript
sum2 = (2) => 1 + 2 //output: 3
```

### Winding Up
So we understand that the whole point of the first function `sum(a)` was to generate another function `sum2(b)` in which the value of `a` was fixed. No matter what value of `b` we give to `sum2(b)`, it will always use the same value of `a` which was used when generating `sum2(b)`. 
```javascript
sum2(5)  //output: 1 + 5 = 6
sum2(10) //output: 1 + 10 = 11
sum2(19) //output: 1 + 19 = 20

```

If we had generated a different `sum2`:
```javascript
const sum2 = sum(10)
sum2(50) //output: 10 + 50 = 60
sum2(90) //output: 10 + 90 = 100
```    

Now coming to the original syntax, all we need to do is call both functions together:
```javascript
// instead of doing the following:
const sum2 = sum(10)
sum2(90) //output: 100

// we can do the following:
sum(10)(90) //output: 100
```
The `sum(10)` statement returns a function (`sum2`) which is immediately called. Imagine these steps like this:
```javascript
sum(10)(90) -> sum2(90) -> 100
``` 

## Why
Once you neatly wrap your head around how the **curried function** works, we can look at why you would want to use it. Let's look at a few cases:

> If you need a non-React example, read Case 2 instead of Case 1
### Case 1: Problem
We will do the following steps in this example:

1. Say we have a **React** application that needs to get some data from an **API**, store this data in a **state** variable, and then **render** it.
2. Say we have a function called `getResponse()` that sends a request to a remote server and returns a `promise`. We will pass the `response` through `JSON.parse()` to get an `array` of strings. 
3. We will then remove a particular string, e.g. `'bla'` from the `array` of strings.
4. We will save the `response` in a state variable for rendering.

### Case 1: Solution
Let's have a look at the code. 
#### Step 1
For step 1, we have a state variable to store the array of strings.
```javascript
const [list, setList] = useState([])
```
#### Step 2
For step 2, we will call `getResponse()`, and then once the returned `promise` resolves to give us a `response`, we will parse the `response` to get an array of strings.

```javascript
getResponse()
.then(response => JSON.parse(response))
```
Let's shorten this code a bit:
```javascript
getResponse()
.then(JSON.parse)
```

#### Step 3
For step 3, we will filter out a string `str`, e.g. `'bla'` from the array by comparing each `element` of the array with `str`:
```javascript {hl_lines="1 5"}
let str = 'bla'

getResponse()
.then(JSON.parse)
.then(arr => arr.filter(elem => elem !== str))
```
We can refactor this code to make it neater. Let's extract the array filtration logic into a separate function that takes the array and returns the filtered array.
```javascript {hl_lines="2 6"}
let str = 'bla'
const removeString = (arr) => arr.filter(elem => elem !== str)

getResponse()
.then(JSON.parse)
.then(removeString)
```
However, this is not good code. The function `removeString` can be **impure** if the value of `str` can be changed by some other function in the code.. Its output depends on a variable `str` that is not one of its parameters. This makes its output **unpredictable** and the function **untestable**. This also makes the following highlighted line difficult to understand. 
```javascript {hl_lines="6"}
let str = 'bla'
const removeString = (arr) => arr.filter(elem => elem !== str)

getResponse()
.then(JSON.parse)
.then(removeString)
```
We don't know which string is being removed by the `removeString` function. We have to scan the code looking for the string.
Let's fix this by making `removeString` a pure function and passing it everything it needs as an argument. 

```javascript {hl_lines="5"}
// impure
const removeString = (arr) => arr.filter(elem => elem !== str)

// pure
const removeString = (arr, str) => arr.filter(elem => elem !== str)
```
Now let's update the call to `removeString` accordingly:

```javascript {hl_lines="5"}
const removeString = (arr, str) => arr.filter(elem => elem !== str)

getResponse()
.then(JSON.parse)
.then(response => removeString(response, 'bla'))
```
Wait! Why didn't we just write the following:
```javascript
.then(removeString)
```
That's because .then() only passes the `array` to our `removeString` function, whereas our `removeString` function now needs **two** arguments: `array`, and `str`. So we pass the second parameter ourselves:
```javascript
.then(response => removeString(response, 'bla'))
```

That's fine, but it's too long. We can use our **curried arrow functions** to shorten it. 
Let's make `removeString` a curried function. It will take a particular string as argument and will return another function that will filter out that particular string from any array. 
```javascript {hl_lines="1 5"}
const removeString = (str) => (arr) => arr.filter(elem => elem !== str)

getResponse()
.then(JSON.parse)
.then(removeString('bla'))
```
Now that's easily readable.
#### Step 4
For step 4, we will save the response in the state `list` variable using `setList`:
```javascript
getResponse()
.then(JSON.parse)
.then(removeString('bla'))
.then(setList)
```
Beautiful!

That should be enough.
However, if you optionally need another example problem (that does not involve React or promises), here is another one:

### Case 2: Problem
Say we have the following two things as input:
```javascript
arr = [1,2,3,4,5]
a = 10
``` 
We want to write a function that takes the above two things as input and returns a new array which is formed by performing the following operations (in order) on the input array `arr`:
1. remove all negative numbers from the array
2. add the number `a` to all of the numbers of the input array, i.e. the resultant array in this case will be `[1+a, 2+a, 3+a, 4+a, 5+a]` 
3. now remove all odd numbers from the resultant array
#### Example
```javascript
// inputs:
arr = [-6, 1, 2, 3, 4, 5]
a = 10

// step 1: remove negative numbers
[1, 2, 3, 4, 5]

// step 2: add a to all elements
[11, 12, 13, 14, 15]

// step 3: remove odd elements
[12, 14]
```

Given any values of `arr` and `a`, how would we write a function named `performOperations(arr, a)` to find the output?

### Case 2: Solution
Let's do step 1 first and remove negative numbers using `arr.filter()`:
```javascript
const performOperations = (arr, a) => {
    return arr.filter(element => element >= 0)
}
```
The above function returns the input array with all negative numbers removed.
Let's do step 2 on the filtered array and add `a` to all numbers of the array:
```javascript
const performOperations = (arr, a) => {
    return arr
            .filter(element => element >= 0)
            .map(element => element + a)
}
```
The above function filters out negative numbers from the input array, and then adds `a` to all elements of the filtered array.
Finally, let's do step 3 and remove all odd numbers:
```javascript
const performOperations = (arr, a) => {
    return arr
            .filter(element => element >= 0)
            .map(element => element + a)
            .filter(element => element % 2 === 0)
}
```

### Solution Refactored
#### Refactoring Step 1
Let's see how we can change step 1. 

We have an `arr.filter()` to filter out negative numbers. What if we were to extract out the arrow function that we are passing to `arr.filter()`? I am talking about the following highlighted function:
```javascript {hl_Lines="3"}
const performOperations = (arr, a) => {
    return arr
            .filter(element => element >= 0)
            .map(element => element + a)
            .filter(element => element % 2 === 0)
}
```

We extract that arrow function into a separate utility function called `isNonNegative` and pass it to `arr.filter()`:
```javascript {hl_Lines="1 5"}
const isNonNegative = element => element >= 0

const performOperations = (arr, a) => {
    return arr
            .filter(isNonNegative)
            .map(element => element + a)
            .filter(element => element % 2 === 0)
}
```

#### Refactoring Step 3
Let's see how we can change step 3. We will return to step 2 and we will use our curried arrow functions there. Meanwhile step 3 can be refactored just like we refactored step 1. We extract out the filtering arrow function:
```javascript {hl_Lines="2 8"}
const isNonNegative = element => element >= 0
const isEven = element => element % 2 === 0

const performOperations = (arr, a) => {
    return arr
            .filter(isNonNegative)
            .map(element => element + a)
            .filter(isEven)
}
```
Let's also remove the curly braces `{}` and `return` keyword from the main arrow function.
```javascript
const isNonNegative = element => element >= 0
const isEven = element => element % 2 === 0

const performOperations = (arr, a) => 
        arr.filter(isNonNegative)
           .map(element => element + a)
           .filter(isEven)
```

#### Refactoring Step 2
Notice how the two filter statements now look so neat and are immediately understandable at a single glance, but the map is longer. Can we extract out the arrow function inside `arr.map()` in the same way?

```javascript {hl_Lines="6"}
const isNonNegative = element => element >= 0
const isEven = element => element % 2 === 0

const performOperations = (arr, a) => 
        arr.filter(isNonNegative)
           .map(element => element + a)
           .filter(isEven)
```

Let's try to store `element => element + a` in a separate function and use it in `arr.map()`:
```javascript {hl_Lines="3 7"}
const isNonNegative = element => element >= 0
const isEven = element => element % 2 === 0
const addNumber = element => element + a

const performOperations = (arr, a) => 
        arr.filter(isNonNegative)
           .map(addNumber)
           .filter(isEven)
```

But that will not work! Why? `addNumber` does not know what `a` is. Hmm... let's pass add `a` to its parameters then:
```javascript {hl_Lines="3"}
const isNonNegative = element => element >= 0
const isEven = element => element % 2 === 0
const addNumber = (element, a) => element + a

const performOperations = (arr, a) => 
        arr.filter(isNonNegative)
           .map(addNumber)
           .filter(isEven)
```

But wait, this will still not work, because javascript's `.map()` function will not pass the value of `a` to our `addNumber(element, a)` function. `.map()` only passes the array `element` to our provided function. 

If only there were a way for us to generate an `addNumber` function with a fixed `a` value, that took only one argument (`element`). Aha! Curried arrow functions!

Let's turn `addNumber` into a curried arrow function:
```javascript {hl_Lines="3"}
const isNonNegative = element => element >= 0
const isEven = element => element % 2 === 0
const addNumber = (a) => (element) => element + a

const performOperations = (arr, a) => 
        arr.filter(isNonNegative)
           .map(addNumber)
           .filter(isEven)
```

So now, we can use `addNumber` to generate a function that takes only one argument (`element`) and this function can be passed to `.map()`:
```javascript {hl_Lines="7"}
const isNonNegative = element => element >= 0
const isEven = element => element % 2 === 0
const addNumber = (a) => (element) => element + a

const performOperations = (arr, a) => 
        arr.filter(isNonNegative)
           .map(addNumber(a))
           .filter(isEven)
```

Done! Our `performOperations` function is now ready:
```javascript
const performOperations = (arr, a) => 
    arr.filter(isNonNegative).map(addNumber(a)).filter(isEven)
```
You can read that one line and know exactly what is happening without giving it a second thought: *array -> filter non negative numbers -> add the number `a` -> filter even numbers*