---
layout: post
title:      "Many to Many Relationship between Tables that Inherit from User Table"
date:       2020-11-02 12:44:06 -0500
permalink:  many_to_many_relationship_b_t_tables_that_inherit_from_user_table
---


For this project we need to have a many to many relationship with a has_many through relationship. I decided to do an application in which tutors can have many students through appointments and vice versa. With both tutors and students being a type of user I had a decision to make about how I wanted to set up the tables and classes in order to have the cleanest/clearest setup possible for my relationships. Here are some options I considered:

**1)**  Don’t bother with a separate user model/table. This will mean that I don’t have to set up an extra relationship, which sounds less messy.

**Problem =>** This will mean that the code will not be DRY, because I will have to write all the "user" code for both the tutors and students tables and controllers (plus another time if I want to add an admin or any other model)

**Conclusion =>** Can’t do it. Gotta keep the code DRY.

**2)**  Set up a User parent class for the tutor and student model to inherit from (class Tutor < User  &&  class Student < User).

**Problem =>** While this cleans up the code in the models, it still doesn’t solve my DRY issues with the tables and controllers, as I will have to repeat user columns in the tables and user functionality in the controllers. Plus, it doesn't allow me to set up unique attributes for the Tutor class and Student class.

**Conclusion =>** Still not DRY enough and lacks some important functionality.

**3)**  Set up a user parent class for tutors and students AND a has_one/belongs_to relationship between these models. This way Tutors & Students inherit attributes from the User class, yet have their own unique attributes. This will also create greater flexibility if I want to add something like an Admin model to inherit basic User attributes in the application. This is what the relationships would look like in the tables and models:  

```
UsersTable
t.string :name
t.string :email
t.string :password_digest

TutorsTable
t.text :resume
t.integer :rating
t.integer :user_id (belongs to user)

StudentsTable
t.text :about_me
t.integer :gold_stars
t.integer :user_id (belongs to user)

class User < ApplicationRecord
has_one :tutor
has_one :student

class Tutor < User
belongs_to :user
has_many :appointments
has_many :students, through: :appointments

class Student < User
belongs_to :user
has_many :appointments
has_many :tutors, through: :appointments
```

**Problem =>** This doesn't work, which I was really bummed to realize. The issue is that when I have the Tutor and Student class inherit from User, it completely nullifies the table structure for their respective tables. The only attributes that can then be ascribed to tutors and students are the ones outlined in the users table (name, email, password)

**Conclusion =>** Doesn't work, on to the next one...

**4)**  Basically, the same as above, except that the Tutor and Student models don't inherit from the User class. This way I can at least set up code to access the relatioships bidirectionally: `tutor.user.name` or `user.tutor.resume`

**Problem =>** This isn't quite as clean as I was hoping to have it. My goal was to set it up to where all the user information for a tutor or student just inherits from the User model/table, and then I could write `tutor.name`

**Conclusion =>** While it's not quite as neat and tidy as I was hoping, it is functional, DRY, and fairly intuitive, so that's what I'm going to roll with:

```
UsersTable
t.string :name
t.string :email
t.string :password_digest

TutorsTable
t.text :resume
t.integer :rating
t.integer :user_id (belongs to user)

StudentsTable
t.text :about_me
t.integer :gold_stars
t.integer :user_id (belongs to user)

AppointmentsTable
t.integer :tutor_id (belongs_to :tutor)
t.integer :student_id (belongs_to :student)

class User < ApplicationRecord
has_one :tutor
has_one :student

class Tutor < ApplicationRecord
belongs_to :user
has_many :appointments
has_many :students, through: :appointments

class Student < ApplicationRecord
belongs_to :user
has_many :appointments
has_many :tutors, through: :appointments

class Appointment < ApplicationRecord
belongs_to :tutor
belongs_to :student
```


**Alternative =>**  don’t set up the the explicit has_one portion of the user/tutor/student relationship, and just add belongs_to to the student & tutor tables/models. This will still keep the code pretty DRY, but I won't be able to access the relationship bidirectionally. So `tutor.user.name` will work just fine, but I can't do `user.tutor.resume`

