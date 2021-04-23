---
layout: post
title:      "RSpec Testing with Rails"
date:       2021-04-23 11:48:36 -0400
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
# ./spec/models/deck_spec.rb

require 'rails_helper'

RSpec.describe Deck, :type => :model do
  it "is valid with valid attributes"
  it "is not valid without a name" 
  it "is valid without a level" 
end
```
Ok, so this is the basic syntax for setting up our model tester. The lines within the method however don't actually conduct any specific test. Instead, those lines in quotations are what will appear as the description for the test. So, when we run `rspec ./spec/models/deck_spec.rb` in the command line we'll see this:
```
Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Deck is valid with valid attributes
     # Not yet implemented
     # ./spec/models/deck_spec.rb:18

  2) Deck is not valid without a name
     # Not yet implemented
     # ./spec/models/deck_spec.rb:19

  3) Deck is valid without a level
     # Not yet implemented
     # ./spec/models/deck_spec.rb:20


Finished in 0.00386 seconds (files took 1.32 seconds to load)
3 examples, 0 failures, 3 pending
```
As you can see, the description is given at each number, but there are no tests implemented. So let's write a test!
```
RSpec.describe Deck, :type => :model do
  it "is valid with valid attributes" do 
    expect(Deck.new).to be_valid
  end
  it "is not valid without a name" 
  it "is valid without a level" 
end
```
Now when we run the tests, we should get one less test pending:
```

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Deck is not valid without a name
     # Not yet implemented
     # ./spec/models/deck_spec.rb:21

  2) Deck is valid without a level
     # Not yet implemented
     # ./spec/models/deck_spec.rb:22


Finished in 0.02104 seconds (files took 0.96134 seconds to load)
3 examples, 0 failures, 2 pending
```
Ok, let's take this up a notch and add a validator to the name attribute in our Deck model
```
class Deck < ApplicationRecord
    has_many :cards
    validates :name, presence: true
end
```
And now when we run the same test, we'll get an error
```
Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Deck is not valid without a name
     # Not yet implemented
     # ./spec/models/deck_spec.rb:21

  2) Deck is valid without a level
     # Not yet implemented
     # ./spec/models/deck_spec.rb:22


Failures:

  1) Deck is valid with valid attributes
     Failure/Error: expect(Deck.new).to be_valid
       expected #<Deck id: nil, name: nil, level: nil, created_at: nil, updated_at: nil> to be valid, but got errors: Name can't be blank
     # ./spec/models/deck_spec.rb:19:in `block (2 levels) in <top (required)>'

Finished in 0.02622 seconds (files took 0.84503 seconds to load)
3 examples, 1 failure, 2 pending

Failed examples:

rspec ./spec/models/deck_spec.rb:18 # Deck is valid with valid attributes
```
Alright, now let's round this out and write out tests for each description we've made
```
RSpec.describe Deck, :type => :model do
  deck = Deck.new(name: "New Deck")
  it "is valid with valid attributes" do 
    expect(deck).to be_valid
  end
  it "is not valid without a name" do 
    expect(Deck.new).to_not be_valid 
  end
  it "is valid without a level" do 
    expect(deck).to be_valid 
  end
end
```
Ok, let's run the test once more, and we should get this:
```
Finished in 0.01515 seconds (files took 0.82945 seconds to load)
3 examples, 0 failures
```
Glorious!
