---
layout: post
title:      "OO Design in a Single Page JS App - a Primer for React.js"
date:       2020-12-10 22:03:10 -0500
permalink:  single_page_javascript_app_using_object_orientation
---


**Context:** 
The challenge for this project is to make a dynamic single (HTML) page web application using vanilla JavaScript to manipulate the DOM and a Rails API server to fetch requests from. 

I decided to do a game because I thought it would be a good challenge to test my programming skills. I stuck with the theme of my previous project, which was to make a host application for a tutoring website, and so I created a memory game. Basically, it boils down to creating two sided flash cards, which can then be implemented in various ways to range from  just practicing one card at a time to being fun and interactive with scoring, different levels,  a timer, etc. 

Since I only had one week to complete the project, I wanted to start with the game and then be able to add multiple features to this application - such as having a practice element, creating and editing sets of cards, etc. Once I built out a basic version of the game, I started organizing the structure of my application to be able to accommodate expansion. I found out later that my design principles essentially mimic the structure of React.js, which I had not been exposed to yet, so if you start to think - ‘why in the world isn’t he just using React?’ - that’s why. 

**The Challenge:**  
Creating a single page web app with multiple different views (a game, practice, create/edit, list of high scores, etc.) posed an interesting problem that required well-thought design techniques. In building out the game portion of the application I had played around with a couple different ways to render and wipe html elements from the page - such as creating and appending elements entirely with vanilla JavaScript, imposing hidden elements and toggling between which were displayed and which were hidden, and then also JS functions that returned HTML blocks as strings. 

Initially, I was creating most of these elements with JS’s DOM manipulation methods, but I found the functions to be quite tiresome and difficult to come back and edit. It was certainly doable, but intuitively it seemed like there ought to be a better way. 

Toggling between display and hidden block elements seemed like a simple solution at first, but as I contemplated all the other features I intended to add to the application it just didn’t seem scalable. 

Ultimately, I leaned more on JS functions returning HTML block elements as strings to be able to render onto the page. In principle this is very similar to React’s JSX components that remain isolated within their own function until called upon to render onto the page. I preferred this methodology over the others in large part because it allowed me to see each HTML/CSS element more clearly than a multiple-step DOM manipulation function, and it also allowed me to work on and edit a certain “page view”, and then just copy and paste it over as a string to be returned. Much more efficient and developer friendly. Here is an example of what it looked like:

```
function decksUl () {
       return `
       <div class="panel row">
           <div class="col s10 push-s1">
               <ul id="decks-ul" class="collection z-depth-2"></ul>
           </div>
       </div> `
   }
 
document.getElementById('panel').innerHTML += decksUl()
```

**The Design:**  
One of the requirements for this project was to implement some level of object oriented design, including the use of JavaScripts ‘class’ structure. I also knew with all the features I would eventually want to add, the application would need to be structured in an intuitive, well-organized manner. After some refactoring and tinkering, I settled on a file structure that is broken down into the following directories: classes & services, models, pages, and htmlFragments.

**Classes & Services:** 
The purpose of these files is to contain JS classes that handle specific types of functionality solely using class-level static methods. These files include: API, SetListener, and Initialize. This is all pretty straight forward - API handles the fetch requests going to and from the Rails API; SetListener handles adding event listeners on HTML elements during a given view change; and Initialize handles the initialization of a couple key items: session and game.

**Models:** 
These are also JS classes, but each of these classes can be instantiated with their own specific attributes and methods. These classes include: Session, Game, Deck, and Card. 

Session, in its current state, is merely a general container to be able to reference the current user and game at a given moment. It can certainly be expanded a bit and correspond with an API table to log session info for the duration and activity of the application’s use. (Essentially, session represents the ‘state’ of the application/component)

Game is used to contain all of the logic needed to run a game - including a timer, scoring, rounds, randomizer, etc. At the conclusion of a game, it consolidates the basic data of the game’s outcome to send to the API as a GameLog, which can be referenced for high scores and such.

Deck and Card classes are instantiated upon the creation of a Game instance and pull from the API for their basic data. There is a simple has_many/belongs_to relationship in the database where a Deck has many Cards and a Card belongs to a Deck. The Card class additionally has some JS logic that creates an HTML element attribute when it is instantiated.

**Pages:** 
These JS classes handle the different “pages” or “views” that the application may display at a given point during its use. Each one has an initialize function that removes the necessary HTML elements from the page, adds new ones, and then runs an API fetch request if needed as well as adding event listeners. 

Additionally, all of these page classes have a parent class of Page, which uses JavaScript’s Prototypal Inheritance structure to pass along the basic functionality of wiping and building the HTML page. Originally I had a Display class that handled these functions, but I was looking for an excuse to use JavaScript’s unique inheritance structure, and all of these classes needed to use those functions anyway.

**htmlFragments:**  
Finally, these fragment classes contain blocks of HTML that can be called upon to render onto the page by one of the Page classes. I broke them up into 2 different classes: SidePanel and Main. These represent two stable HTML block elements that are never wiped from the page, and which allows for easier placement and styling of the elements through this compartmentalization of the page. These two classes contain methods that return an HTML block as a string for simple rendering onto the page.

Essentially, these fragment classes mimic React.js presentational components by mostly just rendering HTML script, while the Page classes mimic container components by handling the logic associated with the given view and passing corresponding arguments to the fragments. Of course there is plenty more going on under the hood of React components than what I have in this application, but their functionality certainly parallels each other.

Here is a code example of what this functionality looks like:

```
class OpeningPage extends Page {
   static init() {
       // buildPage() is from Page parent class
       this.buildPage(this.initialView())
       API.loadMuppets()
   }
 
   static initialView () {
       return {
           panel: [SidePanel.muppetList()],
           main: [Main.kermitWelcome()]
       }
   }
}
```

For the buildPage() method, I pass in an object as an argument. This object contains two arrays as its properties, each one representing the stable HTML elements upon which blocks of code are added and removed. I found this to be the simplest way of adding elements, because it accurately mimics the structure of HTML code anyway. The important thing is that the ordering is correct.

**Conclusion:** 
At the end of the day, this sort of application is best built using React.js, so rather than building on top of it like I originally intended, I’m going to use this basic concept as my next project and code it using React for much of it. 

However, if I were to continue with this application’s code structure using vanilla JS for everything, there are some improvements that I think it would benefit from.

**Areas for Improvement:** 

1) I think the most obvious improvement is to provide each htmlFragment with its own file, and then structure those files under the directories of which HTML block they will go under. In its current state, the structure is fine and fairly intuitive, but realistically it isn’t scalable. If I was to continue building more features and views onto this app, then those two classes would get way too bulky and unruly. 

2) Another area that I think could be improved is the use of Session. Currently I have it set up that almost every Page method takes ‘session’ as an argument in order to keep track of it throughout the use of the application. Intuitively, I know there must be a better way, but I did not come up with a solution that I felt was sufficient prior to wrapping this project up.

