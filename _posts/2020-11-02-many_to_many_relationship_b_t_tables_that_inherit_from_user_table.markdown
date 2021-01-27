---
layout: post
title:      "Controller Inheritance & Different Types of Users with Rails"
date:       2020-11-02 12:44:06 -0500
permalink:  many_to_many_relationship_b_t_tables_that_inherit_from_user_table
---


Context: 
I used single table inheritance to set up my User model, which has an attribute 'type' that assigns a class type (Tutor or Student) to a newly created instance. The idea is that I wanted to have a many to many relationship between Tutors and Students through Appointments (which has a belongs to relationship with both). In the case of this application, both types of Users are going to have the same core functionality (name, email, password, etc), but each will also have their own unique attributes, and (depending on their class type) will have access to a limited scope of the application. For a more in depth look at the setup, here is the previous blog I wrote about this table and model structure:   https://etcusic.github.io/sti_mti_and_different_types_of_users)

Now, onto the controllers. The reasoning behind trying out a controller inheritance was because my different User types were being read by Rails as either Tutor or Student class, which means that when I put in `user_path(current_user)` - Rails would bring up an error because it's trying to find either TutorsController or StudentsController when it checks the class type of my user. In the moment I was able to work around it by typing in something like `redirect_to "/users/#{current_user.id}"`, when I needed to get my routing done, but I found it messy and ugly, and it just felt like I was working *against* Rails rather than *with* it. 

I mulled over some different options and was compelled by the idea of controller inheritance. For one, it bugged me that the urls were `/users` for both types, and I really hated the idea of writing nearly identitcal code for both StudentsController and TutorsController on top of the identical code I would need to write in their views as well. But I digress....

Here is the basic structure of my application's controllers, going through them one by one:

```
ApplicationController < ActionController::Base

  def current_user
    @user ||= User.find_by_id(session[:user_id])
  end

  def logged_in?
    session[:user_id]
  end

end
```

This is definitely not an exhaustive look at the helper methods and before actions that I had set up in the ApplicationController, but you get the idea. As you can see, I'm using `@user` as my user's variable no matter what the class, which allows me to reuse this variable all over the application no matter what the class type of the user is.  (quick note: I'm skipping over the SessionsController because it needed very minimal refactoring) - Ok, now on to the UsersController:

```
class UsersController < ApplicationController
    before_action :current_user, only: [:show, :edit, :update, :destroy]

    def new
        @user = new_user
    end

    def create
        @user = User.new(user_params)
        if @user.save   
            session[:user_id] = @user.id
            redirect_to show_path
        else
            render :new
        end
    end

    def show
    end

    def edit
    end

    def update
        if @user.update(user_params)
            redirect_to show_path
        else 
            render :edit
        end
    end

    def destroy
        current_user.destroy
        session.delete(:user_id)
        redirect_to '/'
    end    

end
```

So, here in the UsersController is where most of the work is being done. As you can see, I've got all my CRUD routes for the different types of users here except for index, which I chose to put in each class's respective controllers. What you may notice missing, though, are the helper methods being utilized here. Those, I put in the Tutors and Students controllers:

```
class TutorsController < UsersController
    def index
        @tutors = Tutor.all
    end

    private

    def user_params
        params.require(:tutor).permit(:id, :type, :username, :password, :resume, :zoom_link)
    end

    def new_user
        user = Tutor.new
    end

    def show_path
        tutor_path(current_user)
    end
end

class StudentsController < UsersController
    def index
        @students = Student.all
    end

    private

    def user_params
      params.require(:student).permit(:id, :type, :username, :password, :resume, :about_me, :level, :gold_stars)
    end

    def new_user
        user = Student.new
    end

    def show_path
        student_path(current_user)
    end
end
```

As you can see, I putdifferent iterations of the same methods (user_params, new_user, show_path) in each controller. That way it stays contained within itself and can provide the appropriate data depending on which controller the operations are happening. Initially I had these methods in the UsersController, but of course they needed conditionals in order to operate correctly. That means more processing, which is not the objective of keeping code DRY. Also, I was looking for a way to set this application up so that it could be expanded if I wanted to add *many* different types of users rather than just two - which would have meant adding more and more conditionals in these methods as it expanded. 

I put a lot of work in trying to come up with the cleanest DRYest way possible to work within the structure of Ruby with this type of setup. There is probably a better way out there to set up multiple types of users with different attributes/capabilities, but ultimately I'm pleased with this setup from the table => model => controller flow. 

As for the views..... there is still work to be done there. Currently there are more conditionals in my views and their helpers than I would prefer. Also, since the routing is ultimately done from the Students and Tutors controllers, I have to have view pages that match each, which means I can't just have a views/users directory where everything goes in and out from. The next step is to clean up these views with less logic and put in some partials that can be reused throughout the different types of users view pages. I'll save that for a different blog post.
