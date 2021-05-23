---
layout: post
title:      "Implementing OmniAuth"
date:       2021-05-21 10:06:11 -0400
permalink:  implementing_omniauth
---


Ok, my last blog post was about setting up OmniAuth from Facebook, so this post will be about actually implementing it within the application. 

First, we'll set up our user resource to give us a table, model, and controller to work with. We'll keep things pretty basic with a `:name`, `:email`, and `:password_digest`. We'll also want to add a `:uid`, which will be the id sent from Facebook identifying a specific user. So, in the command line of our root directory, we can type in:
```
rails g resource User name:string email:string password_digest:string uid:string
```
Now that we've got our base user resource created, we need to add a route to our `/config/routes.rb` file, which will be used to handle the interaction between our application and Facebook's. This will be the callback route that we used when we set things up on the Facebook side of things. We'll also need to create a method in our sessions controller to handle this request. We'll call this method `create_with_omniauth`, which will handle checking to see if the user is already in our database or if we need to create a new user. First, here is the route:
```
get '/auth/facebook/callback' => 'sessions#create_with_omniauth'
```
Now, for the sessions controller. Before getting into the `create_with_omniauth` method, let's set up our private methods. The first will be the `session_params`, which is used to get the params from the browser when a request is sent. Next is an `initialize_session` method, which will be used to attribute the beginning of the session for the user that is signed in. And finally, we have the `auth` method, which will access our `.env` file to get the necessary info from our OmniAuth setup. Here is what our methods will look like:
```
private

def session_params
    params.require(:user).permit(:email, :password)
end

def initialize_session
    session[:user_id] = @user.id
end

def auth
    request.env['omniauth.auth']
end
```
Ok, now on to our `create_with_omniauth` method. There are a number of ways this method can be written, so feel free to do it the way that makes sense to you. There are a couple different ways that a user can be identified by the info sent from FB - name, email, or uid. Personally, I prefer to use the email attribute to check and see if that user already has an account with our application. If they don't, then we create one with the info available, but if they do, then we just pull up their info. Whereas using the `uid` is specific to FB, and so our user may already have an account but didn't originally sign up through FB. So, here is what that controller method ends up looking like:
```
def create_with_omniauth
    @user = User.find_by(email: auth['info']['email'])
    if !@user
        @user = User.create(
				  uid: auth['uid'],
          email: auth['info']['email'],
          name: auth['info']['name'],
					password: SecureRandom.hex(10)
				)
    end
    initialize_session
    redirect_to user_path(@user)
end
```
With our `bcrypt` gem, the user will be required to have a password, so I just use `SecureRandom` to assign a strong password. We can then redirect them to a page where they can set up their own password if they prefer. But we won't worry about that in this blog. We just want to get a base functionality. Now all we need is a link for the user to click. So in our welcome page, we can put a button like so
```
<button><%= link_to 'Log in with Facebook', '/auth/facebook' %></button>
```
And then we can set up a user show page with
```
<h1>@user.name</h1>
```
And voila! This should render our user.
