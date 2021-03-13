---
layout: post
title:      "Recipes, Ingredients, and Normalization"
date:       2021-03-13 00:24:58 +0000
permalink:  recipes_ingredients_and_normalization
---

Context:
Naturally, any developer must do the recipe/ingredients application, so here is a first glance at mine. For a little background - I actually do have a good deal of experience working in restaurant kitchens and managing logistics professionally, so I wanted mine to do more in the way of data tracking and cost analysis. Ultimately, this is more of a supply management application. I include this because the ingredients will need to be associated with more than just recipes, and each category it is associated with will need its own specific quantity, which means I needed to be more intentional with the backend setup.

Models:
Naturally, we have a User, and belonging to the user model we have Pantry, Recipe, and GroceryList models for now. Each of these models belonging to the user will have many Ingredients associated with them, and each of those ingredients will need specific quantities based on which model they’re associated with. For instance, a Pantry object may have a brown rice ingredient with a quantity of 30 (as in ounces or some other unit), and then there could be 2 Recipe objects with a brown rice ingredient - one needs a quantity of 4 and the other a quantity of 6. 

As a supply management system, I want everything to be standardized. There should be one “brown rice” Ingredient, and it should have a set name, unit of measurement, and cost per unit so that we can keep consistent data and easily track expenses. I had considered implementing a Single Table Inheritance structure to account for these ingredients having a different quantity based on its association, but ultimately it would generate a bloated database with lots of duplicate information and be susceptible to anomolies and bugs with the ingredients data. So, we need a better design structure.

Database Design:
Ultimately, what we need is a join table. However, in addition to the foreign keys of the Ingredient and the Recipe it belongs to, we need a quantity field since that is the only non-standardized field at the moment. What we end up having is a RecipeIngredients table with a quantity field and foreign keys of recipe_id and ingredient_id - and the same goes for PantryIngredients and GroceryListIngredients. This is the general table structure:
```
:ingredients_table
t.string :name
t.string :unit
t.float :cost_per_unit

:recipes_table
t.integer :user_id
t.string :name
t.integer :servings
t.text :instructions

:recipe_ingredients_table
t.integer :recipe_id
t.integer :ingredient_id
t.float :quantity

:pantries_table
t.integer :user_id

:pantry_ingredients_table
t.integer :pantry_id
t.integer :ingredient_id
t.float :quantity

:grocery_lists_table
t.integer :user_id

:grocery_list_ingredients_table
t.integer :grocery_list_id
t.integer :ingredient_id
t.float :quantity
```
This table structure is an example of Normalization, which allows there to be a standardized Ingredients table and any attributes that vary will be split out into a different table(s). This means less duplication of data and the standardization should result in a decrease of anomalies and bugs. The tradeoff is that queries require hitting two or more tables rather than a single one, which can slow the processing down, but we’ll address that at a later point. First, let’s finish setting up the associations. The associations are mostly straight forward at this point but with a slight wrinkle. For the sake of time I’ll leave out Pantry and GroceryList, but they follow the same format as Recipe.
```
class RecipeIngredient
  belongs_to :ingredient
	belongs_to :recipe
	
class Ingredient
	
class Recipe
  belongs_to :user
  has_many :recipe_ingredients, 
  has_many :ingredients, through: :recipe_ingredients
  alias_attribute :ingredients_join_info, :recipe_ingredients
  include IngredientProcessor
```
Like I said, mostly straight forward. The RecipeIngredient belongs to a Recipe and an Ingredient, and then a Recipe has many Ingredients through its RecipeIngredient. I've decided to leave out the `has_many`s in the Ingredient since I don't need a bidirectional association at the moment. The wrinkle is that I’ve aliased out the join table models. I aliased them out with the same name for Pantry and GroceryList as well, which allows me to attach the IngredientProcessor module on each of them. This module will handle putting together the composite objects to send in a standardized and readable format.
```
module IngredientProcessor
    def prepare_to_send
        self.attributes.merge!(ingredients: self.ingredients_with_quantities)
    end

    def ingredients_with_quantities
        joins = self.ingredients_join_info
        ingredients = self.ingredients.map.with_index do |ing, i| 
            ing.id == joins[i].ingredient_id ? ing.attributes.merge!(quantity: joins[i].quantity) : "Handle error"
        end
    end
end
```
And here we see the downside of Normalization - extra queries. However, by choosing to break ingredients and joins into their own arrays and process them from there I avoid having to use N + 1 queries. This saves a bit of processing compared to mapping over all the joined ingredients and calling their associated ingredient one by one. It does admittedly slow the process down, though, but it keeps my supply data in a standardized format.

One solution on speed is that I could skip the assembly step on the backend and send the data (currently using React) up to the frontend to be managed by JavaScript. My feeling is that it’s best to keep the construction and deconstruction of these objects all on the backend, but that’s just an instinct.

I’d love to hear other peoples’ opinion on this setup, so if you have any input on structuring this sort of system in a more efficient and intuitive way please don’t hesitate to reach out!



