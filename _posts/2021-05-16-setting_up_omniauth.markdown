---
layout: post
title:      "Setting up Omniauth"
date:       2021-05-16 15:32:24 +0000
permalink:  setting_up_omniauth
---

Ok, going to set up Omniauth in my rails app today.

The OmniAuth gem allows us to use the OAuth protocol with a number of different providers. All we need to do is add the OmniAuth gem and the provider-specific OmniAuth gem (e.g., omniauth-google) to our Gemfile. In some cases, adding only the provider-specific gem will suffice because it will install the OmniAuth gem as a dependency, but it's safer to add both — the shortcut is far from universal.

In this case, let's add omniauth and omniauth-facebook to the Gemfile. Due to recent changes to the omniauth gem we need to add one additional gem to our Gemfile: omniauth-rails_csrf_protection. Once you've added all three gems, run bundle install. If we were so inclined, we could add additional OmniAuth gems to our heart's content, offering login via multiple providers in our app.

Note: If running bundle install gives you an error with installing thin, run the following:


