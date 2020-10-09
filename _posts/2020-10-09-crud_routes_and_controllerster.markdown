---
layout: post
title:      "CRUD, Routes, & Controllerster"
date:       2020-10-09 15:32:11 +0000
permalink:  crud_routes_and_controllerster
---


The aspect of software development that I am most interested in is the structure of a code base and the best practices on how to structure the code as simply and efficiently as possible. I had a bit of trouble initially setting up my controllers and routes in this project, so I figured I’d write about what I learned sorting through its structure.

The most important lesson I learned on this project was how to better organize my code base before beginning to code. For this content management web app project, I think the best place to start is with the basic models and their CRUD methods. For my models, I have:
(this is a fantasy football application where a user can have many teams, and a team has many players)

~ User - CRUD
~ Team - CRUD
~ Player - CR


And here is the basic outline I followed for a model’s controller that requires full CRUD capabilities:


ModelsController < ApplicationController

get ‘/models’
- READ all instances of corresponding model
- erb :'models/index'

get '/models/new'
- take user to page that will allow to CREATE new instance from model
- erb :'models/new'

post '/models/new'
- handle the actual CREATE of the new instance (if all requirements pass, then redirect user)
- redirect to '/models/#{ @instance.id }'

get '/models/:id'
- READ the specific model instance with the corresponding id
- erb :'models/:id'

get '/models/:id/edit'
- take user to page that will allow to EDIT or DELETE a specific model's instance
- erb :'models/:id/edit'

patch '/models/:id'
- handle the actual EDIT of the model's instance (if all requirements pass, then redirect user)
- redirect to '/models/#{@instance.id}'

delete '/models/:id'
- handle the actual DELETE of the model's instance (if all requirements pass, then redirect user)
- redirect to '/models' OR '/website_landing_page' if user profile is deleted



There is certainly some wiggle room on the structure here, but order is important as the verbs 
POST, PATCH, and DELETE are dependent upon GET routes to flow down to them (get '/models/new' => post '/models/new' & get '/models/:id/edit' => patch/delete '/models/:id/edit'). Beginning with this basic template helps to give a basic route structure that adheres to the CRUD principle as a starting point. 

Note: In this project I also separated out all CRUD method routes into different web pages to help visualize the corresponding structure, which helped keeping the routing clear within the application.

I also created controllers that I felt were needed, but that didn't follow the CRUD functionality
of the models I had. These were:
- Application: handles environment with helper methods where all other controllers inherit from, and directs the application to a landing page
- Sessions: handles the logging in and out of the user with code to authenticate and redirect.
- Errors: Receives errors that may occur in the application, communicates the error to the user, and then routes the user where we want them to go.

This degree of separation of functionality allows for more clear structure of the code base at
its outset, which will allow for easier growth as more functionality is added to it.

