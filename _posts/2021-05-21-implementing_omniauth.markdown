---
layout: post
title:      "Implementing OmniAuth"
date:       2021-05-21 14:06:10 +0000
permalink:  implementing_omniauth
---


Ok, my last blog post was about setting up OmniAuth from Facebook, so this post will be about actually implementing it within the application. 






Implementing the OAuth protocol yourself is extremely complicated. Using the OmniAuth gem along with any omniauth-provider gem(s) streamlines the process, allowing users to log in to your site easily. However, it still trips a lot of people up! Make sure you understand each piece of the flow, what you expect to happen, and any deviation from the expected result. The end result should be gaining access to the user's data from the provider in your SessionsController, where you can then decide what to do with it. Typically, if a matching User exists in your database, the client will be logged in to your application. If no match is found, a new User will be created using the data received from the provider.


