---
layout: post
title:      "Rails API on Heroku"
date:       2021-04-05 14:59:49 +0000
permalink:  rails_api_on_heroku
---


Ok, last time we set up a React project with Github pages, and now we’ll create a Rails API to deploy from Heroku. Just like the frontend project, I was looking for the most simple, least fuss setup, and Heroku seems to do a good job of it. 

First step is creating a Rails app with an API flag. I also go ahead and initialize it with a postgresql database. Heroku cannot use the default sqlite in order to run, and needs postgresql instead. There is a way to set up the app to maintain sqlite for development and use postgresql for production, but for simplicity’s sake I just create it with postgresql right off the bat:
```
rails new rails-on-heroku --api --database=postgresql
```
Now that we’ve got our rails-on-heroku (name of the application) created, we need to make a few modifications. First, and most importantly, is setting up CORS. To begin go to the gemfile and uncomment out the cors gem (if you haven’t initialized the app with postgresql, be sure to set up that gem as well). Then, `bundle install`. Now, go over to the `/config/initializers/cors.rb` file and uncomment the code there:
```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
 allow do
   origins '*'
 
   resource '*',
     headers: :any,
     methods: [:get, :post, :put, :patch, :delete, :options, :head]
 end
end
```
Note: you’ll need to manually adjust the origins. Putting `’*’` will allow calls from anywhere, which is simplest to do while we get things set up. This can be changed later to allow only a specific url to interact with the api.

Next, let’s go over to /config/database.yml - at the bottom of the page there is a production section. We need to add an adapter for postgresql:
```
production:
 <<: *default
 database: my_sous_backend_production
 username: my_sous_backend
 password: <%= ENV['MY_SOUS_BACKEND_DATABASE_PASSWORD'] %>
 adapter: "postgresql"
```
Once we’ve made these adjustments, then let’s go ahead and set up some testable elements to make sure we’ve got some basic functionality. My go to is ```
rails g resource Thing words:string
```
This will give us a table, model, controller, provides our basic CRUD routes. Now we’ll `rails db:migrate` and if everything goes through ok, we’ll set up some seed data:
```
Thing.create(words: “Hello, Sir App of Github Pages!”)
Thing.create(words: “I’m from the land of Heroku.”)
```
Run `rails db:seed` and we should be good to go. Let’s get into the console with `rails c` and check to make sure our seed data is there and any routes we want exist. Once we’ve confirmed the data exists, let’s set up our controller to send some data, then we can move onto Heroku.
```
class ThingsController < ApplicationController
   def index
       @things = Thing.all
       render json: @things
   end
end
```
Alright, we should be good to get this set up on Heroku! First, though, make sure your git commits are all caught up 
```
git add .
git commit -m “all set up now”
```
These next steps will assume that you’ve got a Heroku account and are signed in. If not, be sure to do that first. With that said, we’ll create our Heroku app with 
```
heroku create app-name
```
Note: may need to set up Procfile here

Next, we need to migrate and seed our heroku database and push the code!
```
heroku run rails db:migrate
heroku run rails db:seed 
git push heroku master
```
Assuming the build goes through ok, then we’re ready to go back to our frontend and set up a fetch call!

