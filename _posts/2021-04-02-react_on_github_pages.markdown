---
layout: post
title:      "React on Github Pages"
date:       2021-04-02 10:25:24 -0400
permalink:  react_on_github_pages
---


Ok, time to start deploying my projects, so I dug around to figure out the best way to go about this. I looked into Heroku, which I think I'll use to set up my backend, but I wanted to keep my backend and frontend separated and make things as simple and hassle free as possible. My projects are already set up as repos on Github, so moving them onto Github pages seemed like the natural fit. An added bonus is just how simple the process is! This post is mostly just a simple documentation for me to refer back to, but I hope you find it helpful as well!

For this one, I created a new repo to test out its functionality, but it should work with an existing repo as well. So, to start off I createad a repo named `react-on-github`. Then, I created my react app with npm - `npm create-react-app react-on-github`. And now `cd` into the application and connect it with the repo `git remote add origin git@github.com:etcusic/react-on-github.git`. All very standard up to here. 

Next, we need to install Github pages as a dev dependency - `npm install gh-pages --save-dev`. Once that is installed, we need to alter our package.json file so that our application knows where and how to deploy it. At the top level of package.json we'll put `homepage` as a key and then the url as its value. The format is like this:
```
"homepage": "http://{username}.github.io/{repo-name}"
``` 
Then, in the existing `scripts` property, we'll add a `predeploy` and `deploy` 
```
"scripts": {
  //...
	"predeploy": "npm run build",
	"deploy": "gh-pages -d build"
}
```
It's ready to deploy at this point - easy as that! Before deploying, though, let's make a simple counter inside the `<App />` component to confirm that things are working once we do deploy it:
```
import './App.css';
import React, { useState } from 'react';

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>The count on my app: { count }</h1> 
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}

export default App;
```
Ok, let's deploy! First, `git add .` - `git commit` and `git push`. Then run `npm run deploy`. If everything runs through ok, then just visit the url in your browser, and you'll see your application running on the page!

Note:  This is the command we run every time we want to deploy our latest changes. If we're working on the app, we can make commits and push to the Github repo to our heart's content, but it won't make any changes on the actual hosted page until we run `npm run deploy`. 

Next blog, I'll set up a Rails backend on Heroku so that we can be functioning with a backend as well.

