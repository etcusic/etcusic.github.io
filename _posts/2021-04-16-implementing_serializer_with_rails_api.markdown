---
layout: post
title:      "Implementing Serializer with Rails API"
date:       2021-04-16 20:06:00 +0000
permalink:  implementing_serializer_with_rails_api
---


Been spending a bit more time lately on mapping out how to optimize the backend to my Sous Chef application, and I decided to implement a serializer to handle the initialization info for the user. Essentially, a serializer is a service class designated for the logic needed to assemble the API data that is requested. This way we can keep it out of the controller or model and follow the ever important Single Responsibility principal.

To begin we make a PORO (plain old ruby object), which we'll call `UserSerializer`. Next, we'll give it a simple `initialize` method that will accept the user object and create its own instance of it. Then, we need to set up our `to_serialized_json` method, which is where our data specifications will go. Right now I want to send the user's basic personal info plus their supplies, stores, pantry, and recipes. Furthermore, I need the associated supplies/ingredients that goes along with the stores, pantry, and recipes. Let's take a look at the serializer, and then I'll break down the syntax a bit to show what we're getting:
```
class UserSerializer

    def initialize(user_object)
        @user = user_object
    end
    
    def to_serialized_json
        @user.to_json(
            :only => [:id, :name],
            :include => {
                :supplies => {:only => [:id, :name, :category, :sub_category, :unit]},
                :stores => {:only => [:id, :name],
                    :include => {
                        :store_supplies => {
                            :only => [:supply_id, :cost_per_unit]
                }}},
                :pantry => {:only => [:id], 
                    :include => {
                        :ingredients => {
                            :only => [:quantity, :supply_id]
                }}},
                :recipes => {:only => [:id, :name, :servings, :instructions], 
                    :include => {
                        :ingredients => {
                            :only => [:quantity, :supply_id]
                }}}
            }
        )
    end

end
```
First, you may notice that we're calling `to_json` on the user instance, which lets Ruby know what format the data will be structured as. Next, we use `:only => [:id, :name]` so that we're only getting those specific attributes from the database table and bypass any excess info that doesn't need to be sent up to the frontend. After that, we come to `:include => ` which allows us to access any associated data with a given instance. As you can see the pantry is included under the user, and the pantry's ingredients are included under it. And once again using `:only` to signify the specific data we want sent.

There is a key design choice that I'm still not sure about. The ingredients need to be assembled with the supplies, however, if I do that here in the serializer I will be left with tons of n + 1 querries, which will bog the processing down immensely. Ultimately, the answer to this is to set up a hash/object of the user's supplies and use that for assembling the data. Writing out the logic is easy enough, but the issue is where does this logic go? Here are my options outside of utilizing `:include` method as best as I can see them:

1) Set up a Ruby method which creates a dictionary of all supplies, and then map over all ingredients associated with pantry, recipes, etc and assemble them there. This would be easy enough. It would slow down the processing a bit on the backend, but not too horribly. The issue I have with this method is that whenever the user updates a supply, then it will additionally need to update each ingredient on the frontend as well. Not the end of the world, but I don't love it either.

2) Leave the serializer as is, send the raw data needed for the associations (`:supply_id`), and then assemble the data on the frontend. On the one hand, I don't love the idea of the frontend being responsible for assembling the data because it seems like that is asking for glitches. On the other hand, if a single method can be used for ingredient/supply assembly and is properly synched up with React/Redux's lifecycle methods, then it would be a more efficeint way to send and update the associated data.

That's all I've got for now. If you have a strong opinion on how this code should be structured please let me know! I'd love to hear feedback.
