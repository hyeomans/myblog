title: TypeScript for the Back-end - Unit Tests
date: 2016-09-26 22:21:03
tags:
  - typescript
  - mocha
  - unit-tests
---


I love TypeScript, the problem is that I don't use it, I would love to use TypeScript on the back-end. I've worked on several Node.js projects before, and know the pain of a dynamically typed language on the back-end.

I've tried TypeScript before several times: I first did the beginner tutorial and then did a PluralSight course on it but there was always something that made me quit. 

First it was typings and their weird behavior, then was namespaces vs modules, then was import vs require. This time I will give it another try but now I will document my experience through it.

In this post I will try to setup a new project from start, I will delete previous typescript global installation (just for the lolz), and I will try to setup some mocha tests.

Until this point I haven't searched on how to do it, but I imagine that given that TypeScript will "compile" to JavaScript I shouldn't test the TypeScript code directly and rather require the compiled source.

## Removing old TypeScript (1.8)

Skip this if you don't have TypeScript installed.

I'm pretty I could upgrade my TypeScript installation, but I don't want to do it, so I will remove it first:

```
npm uninstall --global typescript
```

## Installing TypeScript 2

My current version of node is:

```
> nvm use 4.5
Now using node v4.5.0 (npm v2.15.9)
```

Now let's install shinny TypeScript:

```
> npm install -g typescript@2
```

Check if it worked:

```
> tsc -v
Version 2.0.3
```

## Bootstrap you environment

Now for the fun part, creating a new project folder, run npm init, install dependencies and devDevependecies.

```
//I love this part
> mkdir typescript-mocha && cd $_
> npm init -y
> npm install --save-dev --save-exact mocha
```

Now open your package.json and make it look like this, I modified the scripts object:

```
{
  "name": "typescript-mocha",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
  "build": "tsc",
  "test": "./node_modules/.bin/mocha"
  },
  "keywords": [],
  "author": "Hector Yeomans",
  "license": "MIT",
  "devDependencies": {
  "mocha": "3.0.2"
  }
}

```

If you try to run `npm test` it will fail (try it, don't take my word). We'll need to create to folders `src` and `test`:

```
> mkdir src && mkdir test
```

Now try to run `npm test` and it should say:

```
>npm test

> typescript-mocha@1.0.0 test 
> /typescript/typescript-mocha
> mocha

0 passing (2ms)
```

It looks like its working, now let's try `npm run build`:

```
> npm run build

```

You will get the whole `tsc` help documentation. At this point I know that I have to add a new file called `tsconfig.json` which will allow me to setup TypeScript and compile `ts` files to `js`.

Before I pasted the code of my new `tsconfig.json` file I discovered that on version 1.6 of TypeScript you can run `tsc --init` and will give you a default json file:

```
> tsc --init
```

I added two new entries under "compilerOptions", `outDir` and `inputDir`, and added "exclude" array:

```
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es5",
        "noImplicitAny": false,
        "sourceMap": false,
        "outDir": "build",
        "rootDir": "src"
    },
    "exclude": [
      "node_modules",
      "test"
    ]
}
```

Now if you run `npm run build` nothing will happen:

```
> npm run build

> typescript-mocha@1.0.0 build 
> /typescript/typescript-mocha
> tsc
```

I think we're ready to test some TypeScript code.

## Adding tests, the TDD way

So [TDD is dead](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html) but I like The Walking Dead so let's resurrect it and give it a try:

```
//./test/helloWorld.spec.js

const helloWorld = require(process.cwd() + '/build/helloWorld');
const assert = require('assert');

describe('Hello world from TypeScript', () => {
  it('should return `hello world`', () => {
    var result = helloWorld();
    assert.equal(result, 'Hello World');
  });
});
```

Now run `npm test`

```
>npm test
....Error: Cannot find module....
```

So Mocha can't find the module, real TDD in action, let's create our first TypeScript module:

```
//./src/helloWorld.ts

export function helloWorld() {
  return "hola mundo"; //In Spanish, remember TDD
}
```

Now we just have to run two commands:

```
> npm run build && npm test
```

And we will get:

```
TypeError: helloWorld is not a function
```

What? So after reading the error several times it looks like our `helloWorld.ts` file exports an object which has a function named `helloWorld`. I wonder if this is part of the ES6 standard, anyway the corrected version of the test would be like this, note that I changed `const hello = require...` and also `var result = hello.helloWorld();`:

```
const hello = require(process.cwd() + '/build/helloWorld');
const assert = require('assert');

describe('Hello world from TypeScript', () => {
  it('should return `hello world`', () => {
    var result = hello.helloWorld();
    assert.equal(result, 'Hello World');
  });
});
```

So we run our tests:

```
> npm run build && npm test
```

And we get:

```
 AssertionError: 'hola mundo' == 'Hello World'
```

Before making it Green, let's do something cool, let's combine our test script:

```
//package.json
...
"scripts" {
  "test": "rm -rf build && tsc && ./node_modules/.bin/mocha"
}
...
```

Now let's fix our module:
```
//./src/helloWorld.ts

export function helloWorld() {
  return "Hello World";
}
```

And run `npm test`:

```
> npm test
...
✓ should return `hello world`
...
```

## Conclusion

Now we have TypeScript compiling our source files and mocha testing those files, but have we win anything?

I'm not an expert on TypeScript, so I would love some suggestion on how to keep going. My next step would be adding Interfaces and probably taking a fresh look into typings. 

What do you think?