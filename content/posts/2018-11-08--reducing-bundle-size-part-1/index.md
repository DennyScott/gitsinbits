---
title: JS Dead Code Elimination
subTitle: Find and remove code that isn't used
category: "javascript"
cover: "dead-code.jpg"
---

* Three techniques to increase TTI
* We should have some examples of reducing the time to interactive
* Should we have a link to a custom made beefy site?
* Should we then take it to a real world example with Zenhub?
* Do some code mods to allow for Zenhub stuff (but save that for another blog)

Let's first talk about front-end bundle sizes. Bundles are usually referring to Javascript and CSS bundles sent from the server to the user's browser on the initial request of the web app. When we discuss it in passing, the primary focus is usually the size of the web apps main Javascript bundle, and how that bundle affects the initial request load time. It affects this by both needing to be downloaded, and to be parsed/evaluated before the web app can become interactive. Lets take a look at that in the browser:

// Image here of time to intereactive.

Time to interactive (TTI) is a good reflection of our bundle size because often our bundle needs to be completey evaluated before a user can properly interact with our web app. Sites with a long TTI often leave users frustrated and annoyed, as pieces of the site have loaded, but they are not actually able to "do" anything yet. The primary ways to handle this problem is to reduce the main payload with code splitting, optimize your javascript with minification, and to reduce the overall payload size with treeshaking. Those last two go hand-in-hand and will be what we investigate today. First, lets look a bit back in history.

//Image of old Website

Website sizes and request load times have been a concern for many years now, just the culprits of slow load times have shifted. Static sites would often be more concerned with the amount of HTML and, most importantly, the size of images than they would about their ability to shrink bundles, as Javascript was used in a much more supplimental capacity. 

As Javascript moved to the forefront of web, with tools like Moo Tools and JQuery, minification tools were developed to reduce css and javascript footprints, and therefore reduce the the inital request load time. Furthermore, we would shift large, commonly used Javascript tools like JQuery to CDN's to reduce load times. Production sites would point to minified versions of these libraries.

Nowadays, with the emergence of Single Page Apps, and a wide array of Javascript libraries and frameworks commonly used to developed these web apps, the ability to shrink the size of Javascript bundles has never been more important. Since javascript is a dynamic language, it can lack some of the tools commonly found in static languages. One of these tools is the use of static analysis for dead code elimination. Dead code elimination usually refers to the removal of code that is unused in our codebase. Lets take a look at an example:

```javascript
function selectClothes(type) {
    if(type === 'shirt') {
        return {
            type: 'shirt',
            amount: '$10',
        }
	}
    else {
        return {
            type: 'pants',
            amount: '$12'
        }
    }
    return {
        type: 'hat',
        amount: '$5'
    }
}

selectClothes('shirt');
```

Lets assume that `selectClothes` is being called from elswhere in the program. In our example, it's impossible to ever get the `Hat` object. Perhaps it was included at one point, but we decided now that users aren't able to buy hats. This often can be the case when code is refactored and changed through the years, and we are left with "dead code". Even though this code is very small, these bits and pieces can certainly add up, and they are increasing the size of our bundle.

Many minifications tools, like Terser, already include dead code elimination. Lets try passing it through Terser to get a bettter understanding of the tool. First, create a folder for this project. Add the code above into an `index.js.` In your terminal, navigate to that directory. You can either `nom init` before the following command, or you can just add the global flag.

```
npm i terser --save-dev
```

Now lets take a first pass with terser. Run

```
npx terser index.js
```

The output should look something like this:

```javascript
function selectClothes(type){if(type==="shirt"){return{type:"shirt",amount:"$10"}}else{return{type:"pants",amount:"$12"}}return{type:"hat",amount:"$5"}}selectClothes("shirt");
```

Our code is minified, but the dead code wasn't removed from the bundle. Whats the deal? Well the default option of terser doesn't include compression, which is where dead-code elimination happens. So let's add it by giving it the `-c` command.

```
npx terser index.js -c
```

And our result should look something like this:

```javascript
function selectClothes(type){return"shirt"===type?{type:"shirt",amount:"$10"}:{type:"pants",amount:"$12"}}selectClothes("shirt");
```

Look how much smaller it is! Not only did it remove the dead code (notice `hat` is completely gone), but it also opimized our code by changing our if into a ternary!

Now, at one point, we did have to pass our code through tools like ternary before releasing our app to the web. But since then, bundling tools like rollup and webpack have taken over the web world, and we can use these system in conjunction with minification tools like terser. In our example we are going to be using webpack, and adding the terser plugin.

So lets add webpack and the webpack cli to our project:

```
npm i webpack webpack-cli --save-dev
```

We'll be adding and moving a couple files here. First, we are going to add a folder called, `dist`. Inside of dist, we will add the index.html file:

```html
  <!doctype html>
  <html>
   <head>
     <title>Getting Started</title>
   </head>
   <body>
     <script src="main.js"></script>
   </body>
  </html>
```

Then we are going to create a `/src` folder at the route of our project, and move `index.js` into it. Last of all, we need to go into our package.json, and remove the `main`property, and instead add `private: true`

```json
{
    "name": "webpack-terser-example",
    "version": "1.0.0",
    "description": "".
    "private": true,
    "scripts": {},
	"author": "",
	"license": "ISC",
    "devDependencies": {
        "webpack": "^4.27.1",
        "webpack-cli": "^3.1.2"
    }
}
```

With this all setup, return to the root of the project, and let's run the webpack cli tool. 

```
npx webpack
```

After that, we can open up `/dist/main.js` to see our new bundled javascript file. Now, remember, this is adding a bundling system to our app, so there's going to be quite a bit of boiler plate added to the file. This is certainly overkill for only having a single Javascript file. But as our web app gets larger, this boilerplate becomes an insignificant amount of code.

So lets look at that bundle!

```javascript
!function(e){var t={};function r(n){if(t[n])return t[n].exports;var o=t[n]={i:n,l:!1,exports:{}};return e[n].call(o.exports,o,o.exports,r),o.l=!0,o.exports}r.m=e,r.c=t,r.d=function(e,t,n){r.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:n})},r.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},r.t=function(e,t){if(1&t&&(e=r(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var n=Object.create(null);if(r.r(n),Object.defineProperty(n,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var o in e)r.d(n,o,function(t){return e[t]}.bind(null,o));return n},r.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return r.d(t,"a",t),t},r.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},r.p="",r(r.s=0)}([function(e,t){}]);
```

You may have a couple questions. First off, we had talked about adding the terser plugin, but this is already minified and compressed. Well that's because webpack automatically adds the terser plugin for minification for you. Also, you may have noticed a warning in your console when we ran webpack. Since we are not specifying an environment, it assumed production. When the environment is set to production, it will perform the minifications and compression for us.

Maybe a even more important question. Where is the code we wrote? Well, since webpack wraps you file with `esmodules` it can do a bit more static analysis on your code. It determined that the code we wrote had no effects on our code base. The code was never called externally, and never used any non-pure functions, therefore, the entire file was dead-code. And that's correct! We never used that code with the actual front-end. The easiest way for us to get around this is to simply add a console log. So lets do that in our index.js.

```javascript
function selectClothes(type) {
  if(type === 'shirt') {
      return {
          type: 'shirt',
          amount: '$10',
      }
}
  else {
      return {
          type: 'pants',
          amount: '$12'
      }
  }
  return {
      type: 'hat',
      amount: '$5'
  }
}


const clothes = selectClothes('shirt');
console.log(clothes);
```

Now when we run our webpack cli again:

```javascript
!function(e){var t={};function n(r){if(t[r])return t[r].exports;var o=t[r]={i:r,l:!1,exports:{}};return e[r].call(o.exports,o,o.exports,n),o.l=!0,o.exports}n.m=e,n.c=t,n.d=function(e,t,r){n.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:r})},n.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},n.t=function(e,t){if(1&t&&(e=n(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var r=Object.create(null);if(n.r(r),Object.defineProperty(r,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var o in e)n.d(r,o,function(t){return e[t]}.bind(null,o));return r},n.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return n.d(t,"a",t),t},n.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},n.p="",n(n.s=0)}([function(e,t){const n="shirt"==="shirt"?{type:"shirt",amount:"$10"}:{type:"pants",amount:"$12"};console.log(n)}]);
```

Our code now exists in the main bundle. Also note, just like before, our desired dead code was eliminated! Lets try abstracting this a bit more. In our index.js, lets update our code to:

```javascript
const buyShirt = () => ({
    type: 'shirt',
    amount: '$10',
})

const buyPants = () => ({
    type: 'pants',
    amount: '$12'
});

const buyHat = () => ({
    type: 'hat',
    amount: '$5'
})

function selectClothes(type) {
  if(type === 'shirt') return buyShirt();
  else return buyPants();
  buyHat();
}

// selectClothes('shirt');
const clothes = selectClothes('shirt');
console.log(clothes);
```

```javascript
!function(e){var t={};function n(r){if(t[r])return t[r].exports;var o=t[r]={i:r,l:!1,exports:{}};return e[r].call(o.exports,o,o.exports,n),o.l=!0,o.exports}n.m=e,n.c=t,n.d=function(e,t,r){n.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:r})},n.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},n.t=function(e,t){if(1&t&&(e=n(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var r=Object.create(null);if(n.r(r),Object.defineProperty(r,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var o in e)n.d(r,o,function(t){return e[t]}.bind(null,o));return r},n.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return n.d(t,"a",t),t},n.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},n.p="",n(n.s=0)}([function(e,t){const n=()=>({type:"pants",amount:"$12"});const r="shirt"==="shirt"?(()=>({type:"shirt",amount:"$10"}))():n();console.log(r)}]);
```

Notice that not only is the call to `buyHat` is removed, but the actual function has been removed as well. Webpack noticed that the call was not used anywhere, and removed the necessary code. If you were to run the same code only using Terser, and not using webpack, you would get the following output. 

```javascript
const buyShirt=()=>({type:"shirt",amount:"$10"}),buyPants=()=>({type:"pants",amount:"$12"}),buyHat=()=>({type:"hat",amount:"$5"});function selectClothes(type){return"shirt"===type?buyShirt():buyPants()}const clothes=selectClothes("shirt");console.log(clothes);
```

So Webpack has removed the unused function, while Terser is unable to do this, why is that? 

This is because Webpack can assume we are using the `esmodule` system. As of Webpack 4, this is the default module system used. Since the `buyHat` function  is not `exported` (more on this later), Webpack knows the only place this is used in the code is in our `selectClothes` function. Once terser removes the dead code from there, webpack knows it's no longer used anywhere in the codebase, and can safely be removed.

#### Migrating to Terser

Zenhub is already using uglifyjs under the hood. The only thing is, uglifyjs does not currently support minifiying ES6 syntax. There was a project created to handle ES, aptly named uglify-es, but the project has since been abandoned. Terser, the previous mentioned tool, was it's successor, and is what is used in Webpack 4 officially. It is meant to have API and CLI compatiablity with uglify-es, but Zenhub hasn't made that transition yet. Although we currently run Webpack 4, the shift to terser has yet to happen. 

As part of this blog I've made this update. The change was as simple as removing the old webpack plugin, and adding the `terser-webpack-plugin`. There was no change in our bundlesize, but it will be important to have these pieces in place as we continue to try and update our bundle.

#### Upgrade Babel

We are also currently using Babel 6. There was a major rewrite to our application when we migrated a large amount of our codebase from Backbone to React. When we made that migration, Babel 6 was the main player, and ES Modules weren't quite as "necessary" as they are today. Because of that, much of our code base still uses the common js module sytax. Although we have not gotten to tree shaking quite yet, we are going to hit a wall in the near future. Tree shaking requires the ES module system to properly analzye our code base (more info on this later).

Babel 6 had a default module type of common js(cjs). This mean, without any configuration, babel 6 will transpile all of our ES module syntax into cjs before webpack has a chance to analyze it. For the sake of seeing early potential bundle reductions, we manually told the preset not to transpile the module syntax. You can do this by setting the preset-env to not modify modules in the `.babelrc`:

```javascript
"presets": [
    ["babel-preset-env", { "modules": false }],
]
```

This did give us the ability to treeshake our code, but we ran into a number of issues where cjs was the expected syntax, and therefore was reporting the dreaded `cannot find property exports of undefined` error. So, rather than focus on fixing this issue in babel 6, I would rather upgrade to babel 7, and sort the issue out there. First, lets 



The idea of noticing a function is no longer being used and stripped from the code base is an important element for another form of dead-code elimination known as Treehshaking.

This is just step one in our initial goal of dead code elimination. What about treeshaking?

Tree shaking is used to describe a process of dead-code elimination, where 