---
layout: post
title:      "Simplified Search Query in Rails"
date:       2021-06-14 17:21:41 +0000
permalink:  simplified_search_query_in_rails
---


**Context:**
My last couple of posts were about setting up a simple search in my Rails app, and I have since revisited that code and revised it to be more clean and efficient. For the search I use a plain old Ruby object (PORO) to create a deck search object, and then use those attributes to finds decks that have matching fields. These fields are `:admin_approved`, `:level`, and `:user_id`. Initially I filtered through one at a time rather than crafting a succinct search. Here is what the original deck search class looked like:
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
        #=>  1) filter_admin, 2) filter_level, 3) filter_user
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
As you can see in the `decks` instance method, each filter method runs independently of the others. At the time when I put this together, I figured there was probably a more succinct, efficient way, but I didn't really have time to dig around and find it. After thinking about it and taking a closer look at ActiveRecord's querying options, I found a way to conduct this search much more cleanly. 

**The query method:**
Basically, all I did was convert the ruby object to a hash using `as_json`, then filter out all empty values within the hash, and finally pass that hash into ActiveRecord's `.where` method to get my criteria to seach through the decks. And here is what I ended up with:
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
        Deck.where(self.as_hash)
    end

    def as_hash
        self.as_json.delete_if{|key, value| value == ""}
    end

end
```
And there you go. Less code and just one query rather than a different filter method for each attribute. Also, it's abstracted out so that this search object could have ten times as many attributes, but `decks` and `as_hash` would still remain the same. Pretty cool!
