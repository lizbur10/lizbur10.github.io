---
layout: post
title:      "BostonEvents: My CLI Data Gem Project"
date:       2017-12-25 18:53:40 +0000
permalink:  bostonevents_my_cli_data_gem_project
---

https://github.com/lizbur10/boston-events-cli-app

The Object Relationships lessons and labs in  the OO Ruby section of the curriculum were challenging for me. Wrapping my head around object relationships and how to get everything to collaborate -- which classes should be responsible for what, how to call a class's methods from another class -- was a struggle. But once I got to my CLI Project I discovered something: if you're building the app from the ground up, it's actually easier. You let the natural structure of your project guide you in how to organize and tie everything together. 

The project I chose allows users to explore events happening around the Boston area. Because the website I was scraping had events organized into categories (stage, music, art, etc.), that was the logical place to start: present users with a list of categories and ask them to choose one.

![](http://burtonux.com/welcome_screen.png)

At the start of the program, the categories are just strings (i.e., the name of the category) that are `puts`ed by the CLI:

```
def puts_categories
    puts "1. Stage"
    puts "2. Music"
    puts "3. Art"
    puts "4. Culture"
    puts "5. Kids"
    puts "6. Top Ten"
end

def select_category
  puts_categories
  input = gets.strip.downcase
  case input
  when "1"
    category_name = "stage"
  when "2"
    category_name = "music"
  when "3"
    category_name = "art"
  when "4"
    category_name = "culture"
  when "5"
    category_name = "kids"
  when "6"
    category_name = "top-ten"
  when "exit"
    abort ("\nThanks for stopping by -- come back often to check out what's going on around town!")
  else
    puts "I'm not sure what you want - please enter a category number or type exit"
    select_category
  end
  BostonEvents::Category.find_or_create_by_name(category_name)
end

```

The first time a category exists in any real way is when the user selects it. Since this happens in the CLI, it makes sense to call the method that creates the Category object there. The #find_or_create_by_name method called at the end of the select_category method checks to see whether the Category exists already and, if not, creates it. The method itself obviously belongs in the Category class:

```
class BostonEvents::Category
  
  attr_accessor :name, :events

  @@all = []

  def self.all
    @@all
  end

  def self.create_by_name(name)
    category = self.new
    category.name = name
    category.events = []
    @@all << category
    category
  end

  def self.find_or_create_by_name(name)
    @@all.detect { | category | category.name == name } || create_by_name(name)
  end

end
```

The class instantiates Category objects, keeps a list of those objects, and initializes an array for each instance that will hold all the events that belong to that instance.

After the user selects a category, the next logical step is to start scraping data about the events in that category. Because the information being scraped is all about individual events, the code obviously belongs in the Event class. The event.rb file is where most of the project's heavy lifting is done: 

1. Scrape each event associated with the category the user chooses
2. Instantiate the Event object
3. For the primitive attributes (name and dates), assign the scraped strings as the Event object's attributes
4. For object attributes (Venue and Sponsor), find or create the Venue/Sponsor objects and assign them as attributes of the Event object using #add_venue/#add_sponsor instance methods.  The #add_venue/#add_sponsor methods also add the Event instance to the list of events that belong to that Venue or Sponsor instance.
5. Assign the category passed in from the CLI as an attribute to the Event object using the #add_category instance method. It isn't necessary to instantiate the category object at this point because that was done earlier in the process, from the CLI.

```
  def add_venue(venue_name)
    venue = BostonEvents::Venue.find_or_create_by_name(venue_name)
    self.venue = venue
    venue.events << self
  end

  def add_sponsor(sponsor_name)
    sponsor = BostonEvents::Sponsor.find_or_create_by_name(sponsor_name)
    self.sponsor = sponsor
    sponsor.events << self
  end

```

As I was working on the code to scrape all the info about each event, it became pretty clear which event attributes should be instances of classes (Venue, Sponsor) and which didn't need to be (name, dates). Name and dates are simple descriptors or characteristics of an event, but events naturally belong to a Venue and Sponsor just as they belong to a Category. Instances of these classes, correspondingly, may have many events.

Creating the code for the Venue and Sponsor classes was extremely easy:  aside from the class and variable names, it was exactly the same as the code for the Category class I had created earlier. The functionality of the three classes is identical -- the only difference is that the venue and sponsor names are scraped while the category name is passed from the CLI, and the classes' methods don't care where the names come from.

In the app as it exists now, the Venue and Sponsor information is assigned to the Event objects but it isn't really used. If I wanted to add, for example, List by Venue or List by Sponsor functionality, the code necessary to write out the events already exists for the Category class and could easily be extended to Venue or Sponsor:

```
def list_events_in_category(category)
  puts; puts "Here's what's happening in the #{category.name.capitalize} category:"
  category.events.each.with_index(1) do | event, index |
    puts "#{index}. #{event.name}, #{event.dates}, presented by #{event.sponsor.name}"
  end
end

```

The tricky part would be making sure that all the events belonging to that Venue or Sponsor are scraped and instantiated before running the corresponding #list_events method. A similar issue would arise with regard to potential List All Events functionality. Because event scraping is initiated by the category the user selects, any events that belong to categories that haven't been viewed by the user won't exist yet.

While the process of completing this project involved many challenges, the organization and structure came together much more easily than I was expecting. The advantages of using an object oriented approach became much more apparent to me as I worked to create the app. One particularly memorable moment was when I changed my mind about including the "Culture" category from the website I was scraping. I initially chose not to because there didn't seem to be much going on on that page, but later I took another look and decided to include it after all. Here's what I had to do to make that happen:

1. Add a culture item to the puts statements on the cli page
2. Update the case statement on the same page to match.
3. Nothing - that was it.

Fun.
