---
layout: post
title:      "Active Record: A Summary"
date:       2018-02-15 19:56:53 +0000
permalink:  active_record_a_summary
---

To get some additional practice with ActiveRecord -- which I was told was going to be important moving forward -- I decided to refactor my CLI project to include a database. The project turned out to be a bit more of an adventure than I was anticipating, but it was very helpful in terms of absorbing and improving my understanding of what ActiveRecord does and why it's important. I will try to summarize what I learned here.

My CLI app scrapes information about events currently going on in the Boston area. The structure of the project and its relationships are as follows:


![](http://burtonux.com/flatiron_blog/boston_events_project_structure.png)


In order to implement ActiveRecord and use a database, there were two main things that needed to happen:


![](http://burtonux.com/flatiron_blog/thing_one.png)


**Thing One: create the database and set up the tables. **

The database is created in the environment.rb file:

```
ActiveRecord::Base.establish_connection({
  :adapter => "sqlite3",
  :database => "db/boston_events.sqlite"
})
```

The database tables are created using migration files which live in the `db/migrate` folder:


![](http://burtonux.com/flatiron_blog/migration_files.png)


For categories and venues, coding the migrations is straightforward: create one column for each of the object's attributes.


```
class CreateCategories < ActiveRecord::Migration
  def change
    create_table :categories do |t|
      t.string :name
      t.string :url
    end
  end
end
```

```
class CreateVenues < ActiveRecord::Migration
  def change
    create_table :venues do |t|
      t.string :name
    end
  end
end
```

For events, which belong to categories and to venues, you also need to include columns for the foreign keys for those objects: category_id and venue_id:

```
class CreateEvents < ActiveRecord::Migration
  def change
    create_table :events do |t|
      t.string :name
      t.string :dates
      t.string :deal_url
      t.string :website_url
      t.integer :category_id
      t.integer :venue_id
    end
  end
end
```

Generally, any object that belongs to one or more other objects will have to have a foreign key column for each object it belongs to.

Once the migration files have been created, you create the tables by running: `rake db:migrate`


![](http://burtonux.com/flatiron_blog/thing_two.png)


**Thing Two: Define the models and the relationships among them**

The files that define the models and relationships are in the `lib/` folder, along with some other program files.


![](http://burtonux.com/flatiron_blog/models.png)


The files themselves look like:

```
class BostonEvents::Category < ActiveRecord::Base
  has_many :events
  has_many :venues, through: :events
end
```

```
class BostonEvents::Venue < ActiveRecord::Base
  has_many :events
  has_many :categories, through: :events
end
```

```
class BostonEvents::Event < ActiveRecord::Base
  belongs_to :category
  belongs_to :venue
end
```

These files define the relationships among the objects and may also include other methods you've written for your app that are associated with the classes.

**The Magic of ActiveRecord**

Once migrations have been run and the class models have been specified, ActiveRecord does its magic:


![](http://burtonux.com/flatiron_blog/activerecord_magic.png)


Why is this important? Well, for starters:
1. we no longer need to write raw SQL commands for many common queries
2. we no longer need to code methods (or create attr_accessors) to do things like return, modify, save, find, or create objects (CRUD actions)
3. ActiveRecord enables us to use method chaining that is semantic and transparent

**ActiveRecord Associations**

Due to a couple layers of abstraction, the [documentation for ActiveRecord Associations](https://apidock.com/rails/ActiveRecord/Associations/ClassMethods) can be challenging to wrap your head around:


![](http://burtonux.com/flatiron_blog/belongs_to_doc.png)


**Belongs to**

This page in the documentation is describing the methods that an object inherits (thanks to ActiveRecord) when its class is identified as belonging to another class. In  my example, events belong to categories, so this page describes the methods that my event objects inherit. These methods are instance methods, so they are called by prefixing the object (the instance of the belonging-to class) in the usual way, using dot notation: `this_event.<method_name>`

The word 'association' in the documentation is simply a placeholder for the "owning" objects. In my example, a particular event instance belongs to a particular category instance. Therefore, if I want to know which instance of category `this_event` belongs to, I construct the method call by replacing 'association' with 'category': `this_event.category`. Events also belong to venues, so the corresponding method call would be: `this_event.venue`. This construction should look familiar: calling `#category` is calling the Category class's getter method. Calling `#venue=` is calling the Venue class's setter method. The abstraction in the documentation can be hard to wrap your head around at first, but the functionality it describes is familiar and intuitive.

Examples:
* `this_event.category` will return the category object `this_event` belongs to
* `this_event.venue=(this_venue)` will assign `this_venue`  as the "owner" of `this_event`
* `this_event.build_category(:name => "Category Name")` will instantiate a new category with the name "Category Name" and assign it to `this_event`

![](http://burtonux.com/flatiron_blog/has_many_doc.png)


**Has many**

Similarly, when you establish a has_many relationship, the "owning" object inherits a whole bunch of instance methods from ActiveRecord. (Only the first few are shown in in the screenshot.) I find the documentation for the has_many relationship to be a little more intuitive: if an object "has many" of another object, it has a collection of them. Therefore it makes sense that the word 'collection' in the documentation stands in for the object that there are many of. As with the "belongs to" relationship, these are instance methods, and you call them by prefixing the object that "has many" of the "collection" objects. For example, calling `this_category.events` will return an array containing all the event instances that belong to `this_category`. 

Note that ActiveRecord also establishes conventions that are semantic: my class names are Category and Event, but I query the events that belong to a category instance using the plural form: `this_category.events`. This is because, by defining the has_many relationship in the model, I told ActiveRecord that there are many of them.

As mentioned above, there are a lot more methods that you get for free when you establish a "has many" association; they are described in detail [in the documentation](https://apidock.com/rails/v4.2.7/ActiveRecord/Associations/ClassMethods/has_many).

To close, I'm going to talk a little about two particularly cool features that ActiveRecord provides.

**#1 Join Models**

My app also contained a couple of join -- or "has many through" -- relationships. Categories have many events, and events belong to venues, so categories have many venues through events (and vice versa). Because of the magic of ActiveRecord, you don't need to do an explicit join a la SQL, and you don't need to "go through" the events they share. In other words, this works: `this_category.venues`. ActiveRecord has told my program that it needs to first find all the events associated with `this_category` using the events' category_id foreign key, then find all the venues associated with those events, this time using the events' venue_id foreign key, and return them. For comparison, the corresponding SQL query is*: 

```
SELECT  "venues".* FROM "venues" INNER JOIN "events" ON "venues"."id" = "events"."venue_id" WHERE "events"."category_id" = ? LIMIT ?  [["category_id", 283], ["LIMIT", 11]]
```

Nice, no?


**#2 Method Chaining**

ActiveRecord also enables method chaining: the program knows how everything is tied together, so you can do things like: `this_category.events.first.venue.name`, which will return the name of the venue associated with the first event that belongs to the category. Or `this_venue.sponsors.find_by(:name => "Sponsor Name").events`, which will find a particular sponsor associated with the venue and return all the events belonging to that sponsor. (Sponsors are making a surprise appearance here: they were in my project but I left them out of the descriptions above to simplify things.) Or `this_venue.events.each{ |event| puts "#{event.name}, #{event.dates}"}`, which puts out a list of all the events happening at a particular venue, along with the dates they're happening. 

Chained methods make for efficient code plus they're semantic and intuitive which makes them a lot of fun to use. After I finished refactoring my project, I got into the console and played around for a while. It was mainly to make sure the examples I give in this post really work, but it was also just playing. I recommend it both as a learning experience and as entertainment.



*you can see the SQL corresponding to any ActiveRecord query by adding this line to the console file: `ActiveRecord::Base.logger = Logger.new(STDOUT)`. I learned how from [this Avi video](https://youtu.be/ZfJ1rqFcNFU).

