title: ES6 Arrow functions
date: 2015-09-21 22:14:24
tags:
  - javascript
  - es6
---

Last night I was reading this post: [ES6 arrow functions, syntax and lexical scoping](http://toddmotto.com/es6-arrow-functions-syntaxes-and-lexical-scoping/) and going through the comments I saw this question:

```
so arrow functions always inherit scope?
```

The answer was by Barney: `always`.

I went to the console and typed:

```
nvm use 4
node
var doSome = () => { console.log(this.x) }
doSome.call({x: 'hello'});
global.x = 'hello';
doSome.call({x: 'good bye'});
```

Could you guess what is going to be printed?

I could replicate this on ES5 without fat arrow function.

```
nvm use 0.10
node
var doSome = function() { console.log(this.x) }.bind(void(0));

doSome.call({x: 'hello'});

global.x = 'hello';
doSome.call({x: 'good bye'});
```

What do you think? What does Babel does?
