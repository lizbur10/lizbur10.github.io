---
layout: post
title:      "jQuery Project Causes Massive Slowdown on the Highway to Graduation"
date:       2018-07-08 12:34:16 +0000
permalink:  jquery_project_causes_massive_slowdown_on_the_highway_to_graduation
---


The Rails project on top of which I needed to build some dynamic jQuery features had evolved over time into a hulking Frankenstein's monster, bristling with obscure nuts and bolts and dragging bits of clanking functionality behind it. I had started way back in the Sinatra section with an already fairly complex model (described [here](http://burtondev.com/sinatra_project)), and then made matters worse by indulging in some attempts at cleverness (described [here](http://burtondev.com/a_rails_project_thats_for_the_birds)). As a result, when the time came to add jQuery on top of it, it felt a little like trying to erect a cathedral on top of a house of cards. Before I could get started with the jQuery part, I had to undo a lot of stuff I'd done previously and get back to a solid foundation. And before I could do that, I had to identify the stuff that needed to be undone and figure out how to undo it. It was overwhelming.

**Slowdown factor #1: Overwhelmed Brain**

So I started drawing diagrams and making lists. I needed to wrap my head around the pieces I wanted to render using jQuery instead of Rails, how everything was currently tied together, and what I needed to re-engineer to get where I was going.

![](http://burtonux.com/flatiron_blog/diagrams_lots_of_diagrams.jpg)

**Slowdown factor #2: Wrapping my head around the task**

First, I had to figure out which pieces I couldn't just build jQuery on top of. One example was how the creation of reports was handled. It started with this screen:

![](http://burtonux.com/flatiron_blog/old-data-entry-form.jpg)

The user selects the date, then adds the first banding record into the table. Clicking Continue does several things: 1) creates the report, 2) creates an instance of birds_of_species (which is the join table: reports have many birds_of_species and have many species through birds_of_species), and 3) redirects to the edit page:

![](http://burtonux.com/flatiron_blog/with-one-bird-added.jpg)

It also creates a *blank* instance of birds_of_species and species (using this nifty/pithy line of code I found after extensive Googling: @report.birds_of_species.build.build_species) which gives the form something to wrap around, resulting in the blank line all ready for the next record. I had felt pretty clever when I'd figured out how to mimic functionality that I would later be creating using Javascript, but now I had to undo all that cleverness and rebuild it. 

For one thing, I had to figure out how to separate the creation of the report from the creation of the instances of birds_of_species, since I only wanted to create the report once but a report could have many birds_of_species. I suppose I could have used find_or_create_by but I decided to make the report creation a separate step instead, mostly because that approach seemed less clunky and more elegant, but also because I could fix a couple of other things while I was at it. So now the first step in creating a report looks like this:

![](http://burtonux.com/flatiron_blog/new-report-create-screen.jpg)

Clicking continue creates the report instance then redirects to this page:

![](http://burtonux.com/flatiron_blog/add_birds.jpg)

Here, jQuery takes over. Once the user enters a banding record and clicks Add More, jQuery creates an instance of birds_of_species, appends the new record to the DOM, and adds a blank line for the next record:

![](http://burtonux.com/flatiron_blog/new-one-bird-added.jpg)

A couple of other things changed in the process. As mentioned above, in the Rails-only version of the app, entering the first banding record created the report and the first instance of birds_of_species and then redirected to the edit page. This was problematic because the process of editing already-entered data got jumbled up with the process of adding additional banding records, which was suboptimal from both a programming and a user experience standpoint. So, in the process of separating the creation of the report from the creation of banding records, I created an add_birds action, separate from the edit action. This not only made more sense separation-of-concerns-wise, it also improved the user experience. In the Rails-only version of adding banding records everything on the screen was editable and it wasn't clear what would happen if the user made changes to existing information *and* added a new record. In the current screen for adding birds (above) the only action that can be taken on that screen is adding a new banding record. In the current screen for editing existing birds (below), the only action that can be taken is editing existing information.  

![](http://burtonux.com/flatiron_blog/new-edit-page.jpg)

**Slowdown factor #3: Fixing things that didn't need to be fixed**

Technically, not all of this stuff really *needed* to be fixed in order for me to meet the requirements of the jQuery project. I could have just created a separate page, thrown together a little html, and created the necessary jQuery there. However, I almost certainly would have learned less from the process and the app definitely would have suffered for it. Whether it was worth such a lengthy detour, I'm not entirely sure...

