---
layout: post
title:      "RSpec Testing with Rails"
date:       2021-04-23 15:48:35 +0000
permalink:  rspec_testing_with_rails
---


I need to redo a project, so I figured this was a great opportunity to begin implementing tests. I decided to use RSpec and keep things very simple. First of all, I needed to add the rspec gem to the gemfile:
```
group :development, :test do
  gem 'rspec-rails', ">= 3.9.0"
end
```
Run bundle install and now RSpec is available to use. Next step is creating a spec directory with a models directory under that, and then we can create a file for a model tester. In this case, we will test the model Deck, which will have name and level attributes. So, creating the file `deck_spec.rb` - and this is where our tests will go for this ruby model:
```
require 'rails_helper'
RSpec.describe Deck, :type => :model do
  it "is valid with valid attributes"
  it "is not valid without a name" 
  it "is valid without a level" 
end
```

