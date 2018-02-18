---
layout: post
title:      "A Derailment Story (Pre-Rails)"
date:       2018-02-18 07:58:58 -0500
permalink:  sinatra_project
---

For my Sinatra project, I chose to address a real-life need for an organization I'm involved with. This was a mistake. 

Okay, not really, but it turned into a lengthy detour from the curriculum. On the plus side, I now have something that may someday be deployed and used, which would make it all worthwhile. But even if it isn't, I learned a lot. Below are a few of the issues I grappled with in creating my app.

**App Functionality and Structure**

The purpose of the app is to handle data entry and report generation for a bird banding station. Each day, the bird bander in charge is responsible for entering a list of all the species banded that day. They then send out an email with a written account of the day and the list of birds banded. Currently, banders are doing the data entry using an Excel spreadsheet, which is not only unwieldy but also very error-prone. I'm hoping that one day the app will centralize and standardize the data entry process, reduce errors in the data, and reduce the amount of work for the banders.

![](http://burtonux.com/flatiron_blog/index_page.jpg)

The structure of the project is:

![](http://burtonux.com/flatiron_blog/project_structure.jpg)

The Bander and Species classes are hopefully self-explanatory. I struggled with the idea of the Report class for quite a while -- it seemed contrived. I initially started programming the app without it, but the code was becoming more and more convoluted so I eventually decided that adding a Report class would be the lesser of the evils.

The Bird class refers to an individual bird, i.e., a banding event. If two individual birds of the same species are banded on a certain date, two bird objects are instantiated. This created a couple of challenges: 1) the app needs to allow banders to enter the total number of each species banded, not enter each bird individually; and 2) the report needs to show the summary value, not individual birds. To accomplish this, birds are instantiated using a simple `n.times` loop:

```
params[:bird][:number_banded].to_i.times do
    bird = Bird.new(:banding_date => params[:date])
    bird.species = find_species_by_code || Helpers.create_species(params[:bird][:species])
    bird.bander = Helpers.current_bander(session)
    bird.save
end
```

The summary table is written out in the erb file by looping through a hash in which the species objects are the keys and the number banded for each species are the values:

```
{#<Species:0x007fdea374b3f8 id: 18, code: "PHVI", name: "Philadelphia Vireo">=>2,
 #<Species:0x007fdea374b290 id: 19, code: "REVI", name: "Red-eyed Vireo">=>4,
 #<Species:0x007fdea374b150 id: 51, code: "COYE", name: "Common Yellowthroat">=>2,
 #<Species:0x007fdea374b010 id: 53, code: "AMRE", name: "American Redstart">=>3,
 #<Species:0x007fdea374aed0 id: 69, code: "BTNW", name: "Black-throated Green Warbler">=>2,
 #<Species:0x007fdea374ad90 id: 97, code: "BUOW", name: "Burrowing Owl">=>1}
```

This results in the following table:

![](http://burtonux.com/flatiron_blog/birds_banded_table.jpg)

The hash is created by a `count_by_species` method, which uses the Active Record associations and method chaining:

```
def self.count_by_species(date_string)
    report=Report.find_by(:date => date_string)
    report.birds.group(:species).count
end
```

I also needed to handle edits to the data, which posed additional challenges. If the bander mistakenly types in a 3 instead of a 2, correcting the error requires that the app delete an instance. To do that, the program calculates `number_change`, which is the difference between the corrected number passed in from the edit form and the number of bird instances in the database. It then completes a loop `number_change` times. If `number_change` is positive the loop adds bird instances, and if it's negative it deletes bird instances:

```
self.count_by_species(date_string).each do |species, count_from_db|
    number_change = passed_params[:species][species.code].to_i - count_from_db
    if number_change > 0
        number_change.times {self.add_bird(species.code,date_string)}
    elsif number_change < 0
        number_change.abs.times {self.delete_bird(species.code,date_string)}
    end
end
```


**Validations, data persistence, and a Flash mystery**

Between the registration, login, and data entry parts of the app, there are a lot of places where I needed to put in validations. For the interface for adding a new bird (i.e., a new banding record):

![](http://burtonux.com/flatiron_blog/add_birds_form.jpg)

The validation code is:

```
if params[:bird][:species][:code] == "" || 
params[:bird][:species][:name] == "" || 
params[:bird][:number_banded] == ""
    flash[:message] = "Please complete all fields"
elsif !Helpers.validate_alpha_code(params[:bird][:species][:code].strip)
    flash[:message] = "Please enter a valid alpha code."
elsif params[:bird][:number_banded].to_i < 1
    flash[:message] = "Number banded must be greater than zero."
elsif (find_species_by_code && !find_species_by_name) || 
(!find_species_by_code && find_species_by_name)
    flash[:message] = "The alpha code and name do not match - please verify"
else
    <body of method>
end
```

Unfortunately, the error messages are not working in all cases. For some screens/routes, they show up fine and for others they don't. I've verified that the code to require and use Rack Flash is added to all the relevant files; I've used a binding to verify that the error messages are getting created and stored in the Flash hash; I've checked whether the difference between the ones that work and the ones that don't is a redirect vs. render issue (it isn't); I've checked whether it's an issue of redirecting to a different controller (nope). Slack, Google, Magic 8 Ball -- all fruitless. I can always code my error messages from scratch (for example, by using the session hash as I did for the issue discussed next), but I'd really kind of like to figure out what's going on. The mystery persists.

Speaking of persistence, another challenge arose from the fact that, when there was an error condition, all the bander's form entries were lost in the redirect. I didn't like that. For example, if the bander is adding a new bird and an error message is triggered, the route redirects to the form and the fields are cleared in the process. The bander would need to re-enter everything. Even worse, not being able to see the entry that raised the error would make it harder for the bander to figure out how to correct it. 

Fixing this problem turned into rather a production. What I wound up doing was creating a session[:temp] hash whenever a form is submitted and storing the params in it. Then, if an error condition occurs and the program redirects back to the form, the following happens: 
1. an if condition in the view checks whether session[:temp] exists
2. if it does, values from session[:temp] are used to populate the form fields 
3. if it doesn't, the form fields are rendered empty
4. once the form has been successfully submitted (without errors), the session[:temp] hash, if it exists, is cleared

The code is neither elegant nor DRY but until I figure out a better way to do it, it seems to be working.

**Restricting Access**

Finally, I needed to 1) keep banders from editing or posting content that doesn't belong to them, and 2) keep people from being able to access content if they aren't logged in. The code for the first issue was a complicated mishmash of error messages, suppressed buttons/form fields, and redirects that I won't go into here. For the second, I did the following:

```
    get '/birds/new' do
        if Helpers.is_logged_in?(session)
            <body of route>
        else
            redirect to '/login'
        end
    end

```

It's simple code and seems to be working fine, but I had to wrap a lot of routes with that code. I suspect there may be an Active Record callback method I could use to make the code more elegant and less verbose. It's on the list of future improvements.
