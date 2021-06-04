---
layout: post
title:      "Creating a Simple Search with Rails"
date:       2021-06-04 19:53:49 +0000
permalink:  creating_a_simple_search_with_rails
---


**Context:**
I'm working on a simple (at least for now) CRUD Rails application that is designed for teachers and administrators to create and edit sets of flash cards for students to use as study resources. Currently my table structure consists of Users, Decks, and Cards. A Card belongs_to a deck, and a Deck belongs_to a User. Also, the `:users` table has an `:admin` boolean, which of course denotes if that user has administrative capabilities, and the `:decks` table has an `:admin_approved` attribute, which means that it has been reviewed and deemed acceptable as adequate study material for students. What I'm implementing in this blog is a search tool that an administrator can use in order to edit decks that don't belong to them, especially for being able to change a deck's `:admin_approved` attribute to true.

**Code Design:**
I wanted to come up with the cleanest, most elegant solution to creating this search tool, so I had to sort through a number of different options. Initially, I was inclined to put the search route and view within the `:decks` resources, but as I was working on it, I felt like it was cluttering things up a bit more than I preferred, so I decided to look into decoupling it. From there, I played with the idea of creating a full on DeckSearch resource with its own MVC components, which would allow me to make `@deck_search` instances and then utilize Rails' built in tools to pass and manipulate through the MVC structure. Unfortunately, in order to pass these instances along through the MVC structure, they need to be saved to the database, which I didn't really want to incorporate at this time. Ultimately, I figured out a way to leverage Rails' MVC design in a way that decouples the search from the `:decks` resource while not having to involve the database in the process.

**PORO with a Show Controller and View:**
For the model, I created a plain old Ruby object (PORO) `DeckSearch` class with `attr_accessor :admin_approved, :level, :user_id`. Then, I created a `DeckSearchesController` with a `show` method, and in my `/config/routes.rb` file I set up the one route: `get '/deck_search' => "deck_searches#show"`. Lastly, I set up a views directory and show file: `/views/deck_searches/show.html.erb` - which will automatically be directed to from the controller thanks to Rails magic. That's the basic MVC file structure needed for this setup, so let's look into the code itself.

**Model:**
I already mentioned the attributes needed for this model. Now I just need to set up an initialize method for creating a deck_search object. I also set up some empty string default values for these attributes, which will come in handy with the form in our view. So, here is what our model looks like at this point:
```
class DeckSearch
    attr_accessor :admin_approved, :level, :user_id

    DEFAULT_ATTRS = {
        admin_approved: "",
        level: "", 
        user_id: ""
    }

    def initialize(attributes=DEFAULT_ATTRS)
        attributes.each{ |key, value| self.send("#{key}=", value) }
    end

    def decks
        # we'll add search logic here later
    end
end
```
**Controller:**
Initially, I was having issues with the controller because I was trying to incorporate an update route in order to update a current search. The tricky part about it was that I couldn't complete the cycle of passing an instance once it got back to the show route. Ultimately, I decided to just stick with *only* the show route and just make a new deck search object each time. This streamlined everything. Then, it was just a matter of checking to see if the paramaters were filled out or not to determine if the default paramaters should be used:
```
class DeckSearchesController < ApplicationController
    before_action :validate_admin
    before_action :initialize_search

    def show
    end

    private

    def deck_search_params
        params.permit(:admin_approved, :level, :user_id)
    end

    def search_params?
        deck_search_params != {}
    end

    def initialize_search
        @deck_search = search_params? ? DeckSearch.new(deck_search_params) : DeckSearch.new
    end

end
```
**Views:**
At this point, the only tricky part is getting the syntax right for the rails form, including blank and pre-selected fields. Here is my code (I used partials for the form and table), which worked out just fine in rendering an effective search form and output table.
```
# /views/deck_searches/show.html.erb
<h1>Search Decks: </h1>
<%= render partial: "search_fields" %>

<% if @deck_search.decks.length == 0 %>
    <h3>Sorry, no decks match that search.</h3>
<% else %>  
    <%= render partial: "deck_table" %>
<% end %>

# /views/deck_searches/search_fields.html.erb
<%= form_tag '/deck_search', method: 'get' do %>

    <label>Approved: </label>
    <%= select_tag 'admin_approved', options_for_select([ ["Admin Approved", true], ["Not Admin Approved", false] ], @deck_search.admin_approved), include_blank: "All" %><br>

    <label>Level: </label>
    <%= select_tag 'level', options_for_select(["1", "2", "3", "4", "5", "6", "7"], @deck_search.level), include_blank: "All" %><br>

    <label>User: </label> 
    <%= select_tag 'user_id', options_for_select(User.all.collect{ |u| [u.name, u.id] }, @deck_search.user_id), include_blank: "All" %><br>
    
    <%= submit_tag "Search Decks" %>

<% end %>

# /views/deck_searches/show.html.erb
<table>
    <thead>
        <tr>
            <td>Deck Title: </td>
            <td>Approved: </td>
            <td>Level: </td>
            <td>Creator: </td>
        </tr>
    </thead>

    <tbody>
        <% @deck_search.decks.map do |deck| %>
            <tr>
                <td><%= link_to deck.name, deck_path(deck) %></td>
                <td><%= deck.admin_approved ? "Yes" : "No" %></td>
                <td><%= deck.level %></td>
                <td><%= deck.user.name %></td>
            </tr>
        <% end %>
    </tbody>
</table>
```
**Search Filter:**
Now we just need to employ some search logic to filter out the decks according to the search. I chose to keep all the methods within the class, but I'm sure a proper stickler would argue that it belongs in a module or as a class method within the Deck class. Also, I chose to link them together through by passing them as arguments. The method nested deepest is run first and then moves outwards. I left a note in the `deck` instance method. Here is what the finished product for the `DeckSearch` class looks like:
```
class DeckSearch
    attr_accessor :admin_approved, :level, :user_id

    DEFAULT_ATTRS = {
        admin_approved: "",
        level: "", 
        user_id: ""
    }

    def initialize(attributes=DEFAULT_ATTRS)
        attributes.each{ |key, value| self.send("#{key}=", value) }
    end

    def decks 
        # 1) filter_admin, 2) filter_level, 3) filter_user
        decks_array = filter_user(filter_level(filter_admin))
    end

    def filter_admin
        self.admin_approved == "" ? Deck.all : Deck.where("admin_approved = #{self.admin_approved}")
    end

    def filter_level(decks_array)
        self.level == "" ? decks_array : decks_array.where("level = #{self.level}")
    end

    def filter_user(decks_array)
        self.user_id == "" ? decks_array : decks_array.where("user_id = #{self.user_id}")
    end

end
```
And there is my simple search tool with it's own MVC structure. Thanks for reading!
