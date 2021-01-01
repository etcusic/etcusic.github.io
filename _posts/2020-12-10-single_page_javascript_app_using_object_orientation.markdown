---
layout: post
title:      "OO Design in a Single Page JS App - a Primer for React"
date:       2020-12-10 22:03:10 -0500
permalink:  single_page_javascript_app_using_object_orientation
---


The challenge for this project is to make a dynamic single (HTML) page web application using vanilla JavaScript to manipulate the DOM and a Rails API server to fetch requests from. 

I decided to do a game because I thought that would present a good challenge to test my programming skills. I decided to stick with the theme of my previous project, which was to make a host application for a tutoring website, and so I created a memory game. Basically, it boils down to creating two sided flash cards, which can then be implemented in various ways to range from  just practicing one card at a time to being fun and interactive with scoring, different levels,  a timer, etc. Once I built out a basic version of the game, and looked at my tangled mess, I decided to take a step back and get to organizing the structure of my application. 

I had a few challenges in particular that needed to be addressed and required better organizing. First of all, I knew I wanted various iterations of the game, which meant that I would need multiple displays depending on what the user had selected (remember, this is a single HTML page web app, so I couldn't route to different pages). Also, I wanted to be keeping track of information during a session - who played what game and what their score was. Plus, I wanted to set up other views like a list of the user's highest, and hopefully I can get to allowing the user to create and edit their own sets of cards as well. So yeah, perhaps a bit ambitious, but I figure it's good practice to set up my applications in a structural way that will allow me to add more features to it down the road. So here is how I structured it:

I divided my Javascript files into 3 distinct sections: Models, Pages/HTML blocks, and Services/Actions.

1) Models: these are set up to keep a smooth transition from API to application as well as maintain good organizational structure. For the models, I set up classes with constructor functions so that I could create instances from API data, and then use those instances throughout the application in order to make the user experience and the tracked data more dynamic. For the games, I've created a class for each type of game, including a practice model, plus a Deck and Card class for the games to use. I also made a User and Session class. I'm not doing any authentication with this application as of yet, but I figured it was good practice to go ahead and associate the game with a user for when I have time to advance the application. The session class was made in order to efficiently track instances and data from beginning to end.

2) Pages/HTML blocks: These classes were constructed to separate out what would be displayed during a given portion of the applications use. For instance, there is a "landing page" where the user selects their avatar (in place of signing in), which require certain options that shouldn't be displayed after a session has begun. Then, there is the "base page" where the user can select which game/set of cards they want to play, or create/edit a new set of cards, or see high scores. Then of course different games are going to have a bit of difference in visual structure. I played around with dynamically creating and removing these HTML elements with Vanilla JS DOM methods, but ultimately I decided to (for the most part) have these class functions return html blocks as strings. That way it's easier to go back and tweak the CSS of different elements when they're displayed that way.

A key component to having these blocks of HTML code added and subtracted from the page is to divide the base HTML page (index.html) into a few different sections that will remain constant. The only difference is which blocks are going into which compartments. For this application, I used 3 different segments: a header, a side panel, and then a main comparment where most of the interaction takes place. I used CSS class identication for this, so that I could more efficiently wipe and maneuver elements during the application's various transitions. This brings us to the most crucial point in the application's structure:

3) Services/Actions: These files are what handle the transitions, and designing them well consolidates the logic of the application and prevents a spaghetti code nightmare. I structured these files as their own distinct class with only class level methods. This both keeps the types of functions in an easy to find file, plus it makes for writing very readable code when calling these class methods (example: `Initialize.landingPage()` ). For now I have 4 main files to handle the different types of actions and transitions:

* class Initialize: This class is primarily used to initialize a "page" - or the specific setup of the view at that moment. I gave all of these class methods that handle pages/views an almost identical setup to give the code's logic and flow continuity throughout its use. This allows to much more easily adjust the code in different areas of the program should a problem arise or if I want to add a new feature to the application. The basic structure of these functions is: 1) wipe the current view from the different sections of the page. 2) Populate those wiped segments with the desired DOM elements for that view. 3) Add event listeners to the items that require it. Here is an example: 

```
class Initialize {

static basePage (session) {
   this.newPromise(Display.wipeAll())
   .then( Display.buildPage(BasePage.initialView(session)) )
   .then( API.loadDecks(session) )
}
```

I set up a newPromise() function at the top of Initialize class in order to handle the ordering of the actions. As you can probably see, each action must be complete before the next one is run. The view can't be built until the current one is wiped or else it might wipe the one it's building during its asynchronous ordering, and then event listeners can't be added until its elements are populated onto the page. Which brings us to the next category.

*  class Display: This class handles the wiping and building of the various segments of HTML. I put a specific wipe and build function for each segment so that I can get at a specific area if the others don't need to be changed for that action. The wipe function handles all elements that are direct children of that particular segment. Then, the build function takes an array of HTML blocks and adds them to that segment. Naturally, the ordering of this array needs to be accurate as it adds one block after the other. Here is an example:

```
class Display {

static wipePanel() {
   document.querySelectorAll('.panel').forEach( node => node.remove() )
}

static buildPanel (nodesArray) {
  nodesArray.forEach( node => document.getElementById('left-container').innerHTML += node )
}
```

This is definitely a heavy handed way of adjusting an HTML page with JavaScript, but these moves specifically handle the big adjustments based on what's happening at the time. The small adjustments will take place within the different game classes themselves (or with a helper function file attached). This way I can quickly create, add, or subtract new features to the application without having to untangle complex transitions. I also added a wipeAll() and buildPage() function that handles all of those events if an action calls for a full reset.

*   class AddListener: This class handles the getting of any element(s) that need event listeners and providing the action function needed. Each function in this class handles only one add listening event, and then the appropriate ones are placed in a setListeners() function that every page class has, which is then called at the end of every page's corresponding Initialize function. And lastly,

*   class API: This handles all of the fetch requests to and from the Rails API. If a GET request is being made, then the setListeners function is usually added at the end of the API function and the API function is called at the end of the Initialize function.
