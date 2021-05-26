---
layout: post
title:      "Tweeking Vanilla-Nested for Rails"
date:       2021-05-26 19:13:55 +0000
permalink:  tweeking_vanilla-nested_for_rails
---


The project I'm working on requires a nested form where a Deck has many Cards, and I wanted the user to be able to add and/or remove as many cards as they want. I was doing some research on how to accomplish this, and it seemed like finding a gem was the best way to go. I tried Cocoon, but I couldn't get it to work for some reason. I found a different gem: vanilla_nested, which seemed like to be more streamlined, so I gave it a shot instead. I ran into a little bug with it, but I was able to get into the code, tweak it, and then it worked great for me, so I'm going to go through how I got it to work.


