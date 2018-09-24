---
layout: post
title:      "How Javascript ES6's `let` gets you out of hoisting trouble"
date:       2018-09-24 16:27:54 -0400
permalink:  how_javascript_es6s_let_gets_out_of_hoisting_trouble
---

**## 1. Hoisting and the trouble with `var` **

Javascript is a bit weird sometimes, and one its quirks is hoisting, which refers to the practice of using a variable in your code *before*  actually declaring it. Here's an example of some odd-looking code that will work just fine:


```
x = "Cronut"

elem = document.getElementById("fake-donut");  
elem.innerHTML = x; 

var x;
```

As you can see, we first assign a value to `x`, before actually telling our program that `x` even exists as a variable. Weird! Javascript, however, will reshuffle this code internally and push the variable declaration to the top. This is why the above snippet does not cause any errors if we run it in console. In other words, JS will 'hoist' `var x`, to produce this: 

```
var x;
x = "Cronut"
// and so on...
```

So far, so good. But take a look at what happens when we use `var` a little carelessly: 

```
function sheepCounter(n) {
  for (var i = 0; i < n; i++) {
	   console.log(`Meeeh, I'm sheep number ${i}`)
	}
	console.log(i)
}
```


If we call `sheepCounter(3)`, what is going to appear on the console? One might think that `console.log(i) `should be well outside the scope of our `for` loop, which we set to terminate at 2 (i.e. n -1). Yet, strangely, we get the following output: 

```
Meeeh, I'm sheep number 0
Meeeh, I'm sheep number 1
Meeeh, I'm sheep number 2

3
```

Thus, even though we might intend to restrict the scope of `i ` to the `for` loop only, it is actually within the scope of the entire `sheepCounter` function, thanks to Javascript's hoisting the variable declaration (` var i` in our example) to the top again: 

```
function sheepCounter(n) 
  var i;
  for (i = 0; i < n; i++) {
	   console.log(`Meeeh, I'm sheep number ${i}`)
	}
	console.log(i)
}
```
## 2. Enter `lets`

With all this unintuitive and difficult to predict behaviour going on, it's easy to see how `var` can cause bugs in our code. Thankfully, Javascript's ES6 gives us a set of neat substitutes for `var`: `let` and `const`.

`Let` in particular is relevant to the `sheepCounter` example above, where the value of the variable needs to be reassigned on every iteration of the `for` loop (`const` would not allow this). If we rewrote the function above using `let `, we'd get the following:

```
function sheepCounter(n) {
  for (let i = 0; i < n; i++) {
	   console.log(`Meeeh, I'm sheep number ${i}`)
	}
	console.log(i)
}
```

Now if we call the function, the line `console.log(i)` will throw an error:

```
Uncaught ReferenceError: i is not defined
    at sheepCounter (<anonymous>:5:14)
    at <anonymous>:1:1
```
And this makes perfect sense: the scope of `i` is restricted to the block in which it is declared (the `for` loop) and does not extend any further (hence it is `not defined` outside the loop). With the `var` keyword, on the other hand, the scope of the variable is the entire function, not just the block, which, as we have seen, can lead to unexpected results.

**## 3. Conclusion **

To sum up: within functions, `let` is scoped to the level of code blocks, while `var` has function scope. In many contexts, `let` will return `not defined` where `var` will return `undefined` or some other value, as in our first example:


```
x = "Cronut"

elem = document.getElementById("fake-donut");  
elem.innerHTML = x; 

let x;
```

Here and elsewhere, the sensible thing will be to declare either `var` or `let` before actually using it. But even with this proviso, when it comes to predictability of behaviour, `let` would seem to have the edge over `var`. 

**## Resources**
* [Learn-Co Hoisting Readme](https://learn.co/lessons/js-hoisting-readme)
* [W3 Schools on Hoisting](https://www.w3schools.com/js/js_hoisting.asp)
* [MDN article on let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
* [Is var dead?](https://wesbos.com/is-var-dead)

