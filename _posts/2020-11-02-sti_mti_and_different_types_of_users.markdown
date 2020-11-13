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

CreateTutorsTable
CreateStudentsTable

class User < ApplicationRecord
 validates :name, :email, presence: true
 has_secure_password
class Tutor < User
class Student < User
```

As you can see, the table contains both Tutor columns and Student columns even though only one or the other will be created per user. You may also notice, there is an extra attribute “type”, which will allow us to specify what kind of object we are creating. With this setup, here are two ways we can create the same object: 

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

One last option that does DRY things up a bit. Rather than using a parent class and table with direct inheritance, we create a `belongs_to` relationship where the child classes `belongs_to` the parent class

```
UsersTable
t.string  :name
t.string  :email
t.string  :password_digest

TutorsTable
t.text :resume 
t.string :zoom_link
t.integer :user_id

StudentsTable
t.text  :about_me
t.integer  :level
t.integer  :gold_stars 
t.integer :user_id

class User < ApplicationRecord
class Tutor < ApplicationRecord   belongs_to :user
class Student < ApplicationRecord   belongs_to :user
```

This actually keeps things pretty DRY. Granted, it’s not ideal in the way that it doesn’t represent reality well since a user IS a tutor or student, not HAS a tutor or student. Also, there is the question of whether this level of object separation could lead to errors or weird edge cases down the line - we'd have to be extra careful with editing and updating since there are actually 2 separate objects at play for a given instance. 

I'll admit it's clunky conceptually, but it would be functional. And honestly, I hate it about as much as the STI and MTI options anyway

![](https://media.tenor.com/images/f203bbd60006dedaaef4c0fae63c7fdd/tenor.gif)


**UPDATE =>**  I decided to go with Single Table Inheritance. One big reason for this is that it emulates the data structure/flow that I was wanting to achieve with a users parent table that different types of users inherit from. With the Multiple Table Inheritance design I wouldn't be able to access different users using the class_type User. 
```
# STI
Student.create(name: "Jack")  #=> creates a User object with type Student that can be accessed through both User and Student classes

# MTI
Student.create(name: "Jill")  #=> creates a Student object that inherits behavior from User class (not accessible through User class)
```
This just means that more logic would need to be added to the User class in the MTI structure in oder to keep the extra logic out of the controllers, but ultimately my code's behavior and structure would be different than having a parent table to work with (this is me naively optimistic that I will have time to figure out how to metaprogram a gem that can give me the behavior I want). I still really hate all the nil attributes within my user objects, but at the end of the day it won't affect a small scale practice project, and the code's logic will be a bit more succinct with the STI foundation.


**UPDATE #2 =>**  Keeping it dry hasn't turned out how I hoped. The issue I'm having is trying to leverage the advantages of the forms (passing instances to and from them). I was trying to pipe the majority of the work through the UsersController into User view pages, and then use logic in those view templates to discern which partial to apply to that template. Unfortunately, the forms in these templates are looking for the class of the user and direct the instance to its corresponding controller. I tried several workarounds, but I've finally given up. Rails and its magic just wasn't intended to handle this sort of thing, so keeping the app DRY is not possible.
