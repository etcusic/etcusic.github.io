---
layout: post
title:      "Setting up OmniAuth"
date:       2021-05-16 11:32:24 -0400
permalink:  setting_up_omniauth
---

Ok, going to set up Omniauth in my rails app today. OmniAuth is a tool that allows us to use authentication providers like Facebook, Google, etc. alongside traditional username and password. This allows for easy and secure access to the application for the user. So, let 's look at what it takes to set it up.

First we'll need to add the gem as well as any provider specific gems. In this one we'll start out with the Facebook omniauth gem. So, in our gemfile we'll add this:
```
gem 'omniauth'
gem 'omniauth-facebook'
```
Next, we'll create an `omniauth.rb` file which is located in the `config/initializers` directory. Within this file we need to create an OmniAuth builder that specifies three things: provider, key, and secret. This builder is a piece of middleware created by OmniAuth used to execute the authentication strategy. Here is what that looks like:
```
#  config/initializers/omniauth.rb

Rails.application.config.middleware.use OmniAuth::Builder do
    provider :facebook, ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
end
```
Note: the `ENV` refers to a global hash to refer to. We can store any key/value pair in this hash, and then stash it somewhere that is hidden from the site of our repo so that others can't easily see it, which would compromise security. We'll come back to this once we get a key and secret from Facebook. In addition to the gem, we need to create a file env file to put the info. 

To get this we'll need to log into the Facebook developer site, create an app there, and then add the product `facebook login`. Then, we'll choose the web option and enter in the site url. Once we've got that, we need to add `Valid OAuth redirect URIs`, which is the default callback endpoint for the omniauth-facebook strategy. Typically, this is the URL with `auth/facebook/callback` tacked on the end. Once we've got that set up, we can head back over to the dashboard and access the key and secret to apply to our app. First, though, let's revisit that `ENV`.

For this next step we need to add the `dotenv-rails` gem to our gemfile. This gem allows us to store and access environment variables in the `ENV` hash in a secure manner. We'll do this in the root directory with `touch .env` and then be sure to add this into the `.gitignore` file as well so that we don't commit our credentials. Now we can paste our App ID and App Secret in the `.env` file like so:
```
FACEBOOK_KEY=############
FACEBOOK_SECRET=*****************************
```
The key will be all numbers and the secret will be a combination of numbers and lower case letters. And there we go, we're all set up to use OmniAuth! If I feel up for it, I'll blog its implementation into the application itself next go round, but at least we've got the credentials set up here.
