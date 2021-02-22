---
layout: post
title:      "Primer for React using Vanilla JS"
date:       2021-02-22 23:38:46 +0000
permalink:  primer_for_react_using_vanilla_js
---


previous blog post: https://etcusic.github.io/single_page_javascript_app_using_object_orientation 
github repo: https://github.com/etcusic/memory-game-interface 

Context: 
When I built out this project originally (single page app with vanilla JS), I didn't really know what React was or how it worked, just that it was a popular front end library. However, having studied good OOP principles, specifically the Single Responsibility Principle, my refactor ended up looking a lot like React's structure. Since then I've learned how to use React and thought I might as well take it a step further into React-ish code as it could come in handy teaching a new front-end coder how to program in JavaScript on their way to learning React.

The biggest thing that led me to this additional refactor was all the `<script>` tags at the bottom of my HTML file. I had modularized my code and broken them out into their own files like a good little programmer, and I was up to like 20. I looked into different script bundlers like RequireJS and Webpack. None of them were as simple and neat as what I hoped to find, but then it occurred to me that maybe I could just export/import my code. This way each file can get exactly the code it needs from other parts of the application without over populating my HTML with script tags.

Now, I've got one script tag at the bottom onf my HTML page:
```
<script type="module" src="scripts/index.js"></script>
```
The `type="module"` denotes that an ES6 module is being used and allows us to use import and export throughout the application. From there, it's just a matter of specifying what functions or variables are being imported and what file they are located in. Here is my index.js:
```
import { OpeningPage } from './pages/openingPage.js'

OpeningPage.init()
```
And here is an example of OpeningPage from my pages/openingPage.js file:
```
import { Page } from './page.js'
import { muppetList } from '../components/muppetList.js'
import { kermitWelcome } from '../components/kermitWelcome.js' 


export class OpeningPage extends Page {

    static init() {
        this.buildPage(this.initialView())
    }

    static initialView () {
        return { 
            panel: [muppetList()],
            main: [kermitWelcome()] 
        }
    }
}

```
There is actually quite a bit more in the OpeningPage class of my application, but I wanted to trim it down for simplicity's sake, and even still there is plenty to unpack. 

Let's begin with `export` - each and every function and variable that is used outside of it's own file needs to have `export` next to it to be available. Hence, `Page` is exported in its file, which allows us to import it into `OpeningPage` and use it as its super class through the keyword `extends` - which means that its an extension of the Page class through JS's prototypal inheritance structure. The Page class holds functions that allow for building and tearing down from the two stable HTML elements where the view changes happen.

Quick note: I decided to use the class syntax for my pages, because I like how the code can be organized and managed within. It has also been widely used in React for state and props management, but that is changing with the use of Hooks. I could have just as easily used pure functions for all this, and I'm sure there is a good argument to have done so.

As for muppetList and kermitWelcome, these are functions return blocks of HTML in string form. They are essentially like a React 'dumb' component - a block of code that renders HTML but does not manage state. These components are used to be concatenated onto the stable HTML elements by loading them into an an array and processed through Page's buildPage function.

And there you have it - a relatively simple version of React's structure using vanilla JavaScript. Now, it's just a matter of building on top of it by adding more 'components' and writing functions to move them around on the page!


