---
layout: post
title:      "Tree Data Structure & Weekly Menus"
date:       2021-03-26 15:15:25 +0000
permalink:  tree_data_structure_and_weekly_menus
---


**Context:**
This entry is another installment on my kitchen management application. This time I'll be covering how I structured my models and tables to handle a weekly menu based on existing recipes using a tree data structure. 

The idea of this application is to be able to handle inventory, manage recipes, and allow for easy food budgeting. Being able to set up a weekly menu allows the user to see how much their food cost should be for the week and more accurately put together a grocery list, which should cut down on wasted product. Basically, the idea is to be able to operate efficiently, and the code structure should reflect that. The tree data structure is a way to structure hierarchical data, which allows for a *whole* object to be broken down into smaller parts that are related to its parent node. This will allow for quick search querries as well as flexibility in making changes without having to alter the structure of the whole.

**Tables:**
The tree data structure is used for organizing hierarchical data, so let's take a look at the hierarchy - weekly menus are the root node and are comprised of several daily menus, which are in turn comprised of several meals, and each meal has a recipe index with a quantity. So, a weekly menu is the root or parent node, which has many daily menu child nodes, which have many meal nodes. Let's take a look at the tables:
```
weekly_menus table
    t.integer "user_id", null: false
    t.date "start_date"
    t.date "end_date"
    t.index ["user_id"], name: "index_weekly_menus_on_user_id"
end

daily_menus table
    t.integer "weekly_menu_id", null: false
    t.date "date"
    t.index ["weekly_menu_id"], name: "index_daily_menus_on_weekly_menu_id"
end

meals table
    t.integer "daily_menu_id", null: false
    t.integer "recipe_id", null: false
    t.integer "quantity"
    t.string "category"
    t.index ["daily_menu_id"], name: "index_meals_on_daily_menu_id"
    t.index ["recipe_id"], name: "index_meals_on_recipe_id"
end

recipes table 
    t.string "name"
    t.integer "servings"
    t.text "instructions"
end
```
I included the recipes table because it is connected with the meals table, which essentially acts as a join table for recipes and daily menus, which allows that data to be related as well. The meals table also carries a quantity attribute which accounts for how big a batch of the recipe will be made and a category, which will be something like a dinner, lunch, breakfast, second breakfast, elevensies, etc.

**Models:**
Now let's take a look at the models where we put our business logic that will bring all of these things together to be sent up to the end user. Note: as a design choice I decided to create a standardized instance method for each layer of the hierarchy called `send_info` - this method will put together its own object in the way that we want it sent to the user. Let's begin with the bottom of the hierarchy and work our way up.
```
class Meal < ApplicationRecord
  belongs_to :daily_menu
  belongs_to :recipe

  def send_info
    info = { 
      recipe_id: self.recipe_id, 
      quantity: self.quantity, 
      category: self.category 
    }
  end

  def blank_meal
    info = {
      recipe_id: 0, 
      quantity: 0, 
      category: "--"
    }
  end

end
```
Very straight forward here. We just set up a hash for the instance to draw from, which will be easy to use on our frontend. I include just the `recipe_id` rather than bringing the whole recipe along because I want the user to be able to manage recipes on their own, and so they are available to be grabbed easily on the frontend and applied if we need to show or edit our meals. I also include a blank meal class method to send if a daily menu doesn't have any meals yet. Now for the next layer of our hierarchy, which is the daily menu:
```
class DailyMenu < ApplicationRecord
  belongs_to :weekly_menu
  has_many :meals
  has_many :recipes, through: :meals

  def send_info
    daily_menu = {
      id: self.id,
      date: self.date.strftime("%A, %b %d"),
      meals: self.get_meals
    }
  end

  def get_meals
    self.meals.length > 0 ? self.meals.map{ |meal| meal.send_info } : Meal.blank_meal
  end

end
```
As you can see it basically follows the same pattern as the Meal model except it also has a *get_* instance method to retrieve the standardized data from it child nodes. So now, let's take a look at the top of the hierarchy:
```
class WeeklyMenu < ApplicationRecord
  belongs_to :user
  has_many :daily_menus
  has_many :meals, through: :daily_menus

  def send_info
    info = {
      id: self.id,
      start_date: self.start_date.strftime("%A, %b %d"),
      daily_menus: self.get_daily_menus
    }
  end

  def get_daily_menus
    self.get_daily_menus.map{ |menu| menu.send_info }
  end

  def self.this_weeks_menu(user_id)
    menu = self.find_by(user_id: user_id, start_date: self.find_last_sunday)
    menu.send_info
  end

  def self.find_last_sunday
    Date.today.strftime("%A") == "Sunday" ? day : self.find_last_sunday(day - 1)
  end

end
```
The `send_info` and `get_daily_menus` follow the same convention as daily menu, and then I've also added a class method to get `this_weeks_menu` which I use for the user's initialization data. Basically once the user logs in, some basic info will be sent up for the user to access, including the menu for this week. It just needs a simple helper method `self.find_last_sunday` that uses recursion to find the first day of this week, and then use that info to find this week's menu. I'll probably move this to a module later for managing dates, but it made sense for now to keep it here for easy reference. 

There is one more level - `User`, which is technically at the top of the hierarchy. That model has a similar instance method `send_initialization_info`, which as I mentioned before will send some standardized data once the app is initialized. From the user's point of view, though, the weekly menu is at the top.

And that's pretty much it for now. More logic will need to be added as I develop forms for creating and updating my menus and meals. I can also add another layer or two to this data tree if we wanted to break down and manage data from a monthly or yearly perspective. In that case we would just follow the same patterns shown by the existing tree model, which should make querying and managing the data pretty simple and straight forward.

Thanks for reading!
