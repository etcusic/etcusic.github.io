---
layout: post
title:      "Nested Forms in Rails"
date:       2021-03-07 22:45:22 +0000
permalink:  nested_forms_in_rails
---


Context: 
Ok, so I'm doing a little recipe management system for a project. I know, everyone has done this a thousand times. I'm hoping to take mine to a bit of higher lever, though. I used to manage kitchens and even worked as a sous chef, so the end goal is to have a basic kitchen supply management system that I can use not just to store recipes, but also to track the raw cost of ingredients and recipes as well as generate weekly menus and grocery lists. As I develop this application, I'll put the subsequent blog posts right here:

- no additional posts yet -

This will be very backend heavy with some interesting associations, but for now let's look at a basic frontend rails form that involves an association - recipes with ingredients. Naturally a recipe `has_many` ingredients and an ingredient belongs_to a `recipe`. Now, we just need to make a quick form for a new recipe in order to get things kicked off, so let's take a look at the recipe model and controller:
```
class Recipe < ApplicationRecord

  belongs_to :user
  has_many :ingredients, dependent: :destroy
  accepts_nested_attributes_for :ingredients

end

class RecipesController < ApplicationController
    
		def new
        @recipe = current_user.recipes.build
        10.times do
            @recipe.ingredients.build
        end
     end    
		
end
```
First off, we need to establish in the model that Recipe `accepts_nested_attributes_for`. We won't bother ourselves with the rest of the controller just yet and focus on setting up the form. Naturally, we set up our new variable `@recipe` to belong to the current user. Then, we need to generate some new ingredients that belong to this new recipe. In this example, I've just set up an even 10. This is rather clumsy, and I hope to come back and make this more precise and elegant, but in the interest of moving forward it will do for now. On to the view's erb file:
```
<%= form_with model: @recipe, local: true do |form| %>  

    <%= form.label :name, "Title: " %>
    <%= form.text_field :name %>

    <%= form.label :servings, "Servings: " %>
    <%= form.number_field :servings %><br><br>

    <%= form.fields_for :ingredients do |inner_form| %>

        <%= inner_form.label :ingredient, "Ingredient: " %>
        <%= inner_form.text_field :name %>
        <%= inner_form.number_field :quantity %><br><br>

    <% end %>

    <%= form.label :instructions, "Instructions: " %>
    <%= form.text_area :instructions%><br><br>

    <%= form.hidden_field :user_id, value: @recipe.user_id %>
    <%= form.submit "Create New Recipe" %>

<% end %>
```
Here we have the recipe's form, which has its title, servings, and instructions fields. Then, inside this form we nest the ingredients with an inner form. Because all the new ingredient objects are the same, I only need to put one set for a new ingredient and then rails generates the other 9. Now a user can bring up the page, insert the recipe and ingredient information and the submit button will send a 'POST' request back to the controller, so let's go back there and see what we need to do.

First, we'll set up the `recipe_params` which will permit it's own attributes, plus its nested attributes. This is achieved by including `ingredients_attributes: [:name, :quantity]`, which will cycle through each new ingredient from the form and its attributes. Then, we'll pass those recipe_params through `Recipe.create` and it will not only generate the new recipe, but also generate new ingredients associated with it. Here is the controller:
```
class RecipesController < ApplicationController
    
		def new
        @recipe = current_user.recipes.build
        10.times do
            @recipe.ingredients.build
        end
     end    
		 
		 def create
        new_recipe = Recipe.create(recipe_params)
        redirect_to user_recipes_path(new_recipe.user)
    end
    
		private

    def recipe_params
      params.require(:recipe).permit(
			:user_id,
      :title,
			:servings,
			:instructions,
      ingredients_attributes: [ :name, :quantity ]
    )
    end
		
end
```
At the moment, there is a bit of an issue with our setup. Each time we create a new recipe, we are also creating new ingredients. Theoretically, these ingredients already exist in the database. Checking for duplicates can be solved by implementing an `ingredients_attributes=(ingredient)` method which will use `Ingredient.find_or_create_by(:name)` to check for a duplicate. However, each ingredient needs to have its own quantity associated with that particular recipe as well, so we'll forgo implementing this at the moment. Ultimately, this will require some join tables and some trickier associations - ingredients will belong to a Pantry as well, and some recipes (like sauces) will also be an ingredient!

Tackling these backend associations will be next task and most likely my next blog post as well. Stay tuned!

