---
layout: post
title:      "STI, MTI, and Relating Different Types of Users"
date:       2020-11-02 21:25:30 -0500
permalink:  sti_mti_and_different_types_of_users
---


For this project we need to have a many to many relationship with a has_many through relationship. I decided to do an application in which tutors can have many students through appointments and vice versa. With both tutors and students being a type of user I had a decision to make about how I wanted to set up the tables and classes in order to have the cleanest/clearest setup possible for my relationships. For a base user model, I wanted there to be a name, email, and password_digest, and then tutors & students would have several of their own unique attributes ascribed to their models. There are a number of ways to set this up as models and tables, so I had to sort through them to find the clearest and DRYest way possible.

The problem =>  To me, the cleanest/DRYest approach was going to have a User parent class and table from which Tutors & Students would inherit. Something like this:

```
Class User < ApplicationRecord  #(with user validations set up)
Class Tutor < User
Class Student <User
UsersTable => name:string, email:string, password_digest:string
TutorsTable => resume:text, zoom_link:string, rating:integer
StudentsTable => about_me:text, level:integer, gold_stars:integer
```

Unfortunately, this does not work. It would appear as though ActiveRecord is not smart enough to simply allow Tutors & Students to inherit the User info. A big part of this is that when a model inherits from a parent class (Tutor < User), then ActiveRecord will not acknowledge the child’s table and only recognize the parent’s table. So, if I do Tutor.create(name: “Joey”, rating: 4) - it will come back with an error that it does not recognize “rating” as a Tutor attribute. It reads “name” from the User table just fine, though. There is a way to keep the parent class from nullifying the child class with “abstract_class” - however, if the parent class is designated as abstract, then it cannot carry a corresponding table with it.

Bleh.

So then, how do I work around this issue? Well, there are many different ways of doing it, so we’ll look at a few and weigh the pros and cons. Primarily, we will be looking at a Single Table Inheritance option and a Multi Table inheritance option.

The Single Table Inheritance Model is best described as consolidating all the attribute info of similar models into one table, and then distributing those attributes selectively according to the type of class that gets instantiated. For my Users/Tutors/Students example, it would look something like this:

```
CreateUsersTable
t.string :type
t.string :name
t.string :email
t.string :password_digest
t.text :resume  #=> tutor attr
t.string :zoom_link  #=> tutor attr
t.text :about_me  #=> student attr
t.integer :level  #=> student attr
t.integer :gold_stars  #=> student attr

class User < ApplicationRecord
 validates :name, :email, presence: true
 has_secure_password
class Tutor < User
class Student < User
```

As you can see, the table contains both Tutor columns and Student columns even though only one or the other will be created per user. Also, the "tutors" and "students" tables aren't even necessary to build - just their models that inherit from User (which is kinda cool). You may also notice, there is an extra attribute “type”, which will allow us to specify what kind of object we are creating. With this setup, here are two ways we can create the same object: 

```
Student.create(name: "First Last", email: "student@mail", level: 5)
User.create(type: Student, name: "First Last", email: "student@mail", level: nil)
#=> <Student id: 1, name: "First Last", email: "student@mail", password: nil, zoom_link: nil, resume: nil, about_me: nil, level: 5, gold_stars: nil>
```

Note: the object shows up as an instance of Student, but it can also be accessed via User. If you were to type in `User.all` the Student object will show up in the array like that.

This certainly does solve the problem, and I have a functional inheritance, but I really don't like all of the extra attributes that every User instance will be carrying around as nil. Not only does it not appease my code OCD, but it can also get heavily bogged down if I decide to expand the types of users into different types/roles. So let's look at a Multi Table Inheritance Model to see if we can clean things up......

I would describe an MTI as essentially the inverse of an STI. The STI has all of the various attributes at the very top of the model tree, and then distributes them down according to type, but the MTI doesn't hold any of that table info at the top (in this case User), and instead separates each table with its specific functions. In order to accomplish this, though, we need to specify that the parent class is "abstract" - which means that it won't have a corresponding table, and can only pass down other functionality like methods and validations

```
TutorsTable
t.string :name
t.string :email
t.string :password_digest
t.text :resume  
t.string :zoom_link  

StudentsTable
t.string :name
t.string :email
t.string :password_digest
t.text :about_me 
t.integer :level  
t.integer :gold_stars   

class User < ApplicationRecord
 self.abstract_class = true
 validates :name, :email, presence: true
 has_secure_password

class Tutor < User
class Student < User
```

As you can tell, this is not DRY as we are repeating :name, :email, and :password for each type of user. Not great. It's definitely more code here on the setup, but as we think of the application growing and adding more types of users, at least each type of user will ONLY have the attributes that are associated with its model rather than carrying a bunch of nil values around from a bloated User table. I hate repetitive code, but I think in this case I'll go with the Multi Table Inheritance setup, because even though it looks more repetitive on the setup, I believe that down the road it will save a lot of memory and speed with the database since its instances won't be carrying extra nil attributes along with them.

I did also look into the option of User and Tutor/Student being completely separated out and then setting up a belongs_to relationship from Tutor & Student so that they would have to access their name, email, and password through the relationship `tutor.user.name` - but this level of dissassociation seemed like it would be problematic and possibly toss a wrench into the database at some point. I didn't feel like it was worth even setting up the code examples.

And then the last option would be to do some metaprogramming. Which is very tempting as I find these associations to simply not be clean enough for something that seems so commonplace and basic as a simple Table/Model inheritance scheme. To be continued???
