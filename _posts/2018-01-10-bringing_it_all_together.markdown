---
layout: post
title:      "Bringing it all Together"
date:       2018-01-10 19:41:00 -0500
permalink:  bringing_it_all_together
---

Learning to code sometimes feels a bit like learning to juggle. You start off with one ball and toss it up in the air a whole bunch of times until that starts to feel natural. Then you add a second ball and everything goes to hell. But you keep practicing and pretty soon you feel like you can handle two balls, then three, and before long you start to feel like you're getting to be a pretty good juggler. Then you hit a lesson that feels like someone tossed a bowling ball at you.

The ORM section was a juggling challenge for me. Just SQL was bad enough with its rigid syntax and the merry guessing game of what counterintuitive stew of commands would get it to cough up the goods. But then we had to add SQL interactions with a database into the mix along with the object relationships, collaborating objects, etc. of OO Ruby, and it was at least one too many balls for me. By the time I got to the Bringing It All Together Lab I knew I had to do something to try to tame the cognitive overload. So I created this:

![](http://burtonux.com/flatiron_blog/bringing_it_all_together.png)

This diagram shows how I set up the relationships among the methods in the Dog class. Each rectangle or oval represents a method. Methods shown as rectangles act upon the database, while the ovals act upon an object. The create method acts upon both so it's represented by a rectangle with rounded corners. Finally, the yellow methods are instance methods while the white ones are Class methods. 

The ::create_table and ::drop_table methods are quite straightforward; they interact only with the database and only with the dogs table as a whole. The only other methods that act upon the database directly are #update and #save. They both write information to the database, about instances (rows) that either already exist in the database (#update) or don't (#save). 

::find_by_id, ::find_by_name, and ::new_from_db all access information from the database and use that information to instantiate an object. ::find_or_create_by has the same behavior, but also saves the object to the database by calling the ::create method. ::create is the only method that acts directly upon both the database and the object.

I created the diagram to help me keep straight exactly what each method should be able to do. The methods to be implemented were familiar but different -- a recipe for confusion. Some examples:
* It isn't immediately obvious from the names of the #save, ::create, and #update methods whether it's an object or a row in the database that's being saved/created/updated. 
* The find methods find instances that already exist in the database, but they also instantiate an object, which is unfamiliar behavior.
* In earlier sections of the curriculum, the purpose of save methods was to add objects to an array. However, the addition of databases to the picture makes the @@all array irrelevant. But databases can't actually take the place of the @@all array because the objects themselves can't be saved into a database -- only their attribute values can. As a result, the find methods are as much about creating an object as finding one.

Just when you think you've got it, everything changes.

