---
layout: post
title:      "A Rails Project That's for the Birds -- but in a Good Way"
date:       2018-04-11 18:13:26 -0400
permalink:  a_rails_project_thats_for_the_birds
---

For my Rails project I continued working on an app I had developed for my Sinatra project. The purpose of the app is to centralize and streamline data entry for a bird banding station I'm involved with. The project is described more fully in my [blog entry for the Sinatra project](http://http://burtondev.com/sinatra_project).

## A nesting rats' nest 

The structure of the project is:

![](http://burtonux.com/flatiron_blog/project-model.jpg)

Each report is associated with a particular date, and consists of a list of all of the species banded along with the number of each. The relationship is much like the example of Recipe and Ingredients used for one of the Flatiron School labs: the number of birds banded of a given species is an attribute of the join table, birds_of_species (like recipe_ingredients), which belongs to Species and to Report. 

The data entry form looks like this:

![](http://burtonux.com/flatiron_blog/data-entry-form.jpg)

However, my project has an extra layer of complexity: the form was created by using form_for, which was wrapped around an instance of the Report model, but the only field in the form associated with the Report model is date. Alpha Code and Species Name are attributes of Species, and Number Banded is an attribute of BirdsOfSpecies. This made for a somewhat complicated nested form:

```

<%= form_for report do |f| %>
	<h2><%= f.label :date %>: 
	<%= f.date_field :date %></h2>
	<h2>Add Birds Banded:</h2>
		<table>
		<th>Alpha Code</th><th>Species Name</th><th>Number Banded</th>
		<%= f.fields_for :birds_of_species do |b| %>
				<%= b.hidden_field :bander_id, :value => current_bander.id %>
				<%= b.fields_for :species do |s|%>
						<tr>
								<td><%= s.text_field :code %></td>
								<td><%= s.text_field :name %></td>
								<td><%= b.number_field :number_banded %></td>
						</tr>
				<%end%>
		<%end%>
		</table>

	<%= f.submit "Continue" %>
<%end%>
```

The form consists of three layers: fields for species, which are nested inside fields for birds_of_species, which in turn are nested within the form for report. The strong params method wound up looking like:

```
def report_params
	params.require(:report).permit(
		:id,
		:date, 
		:bander_id,
		:content,
		:birds_of_species_attributes => [:bander_id, :number_banded, :id,
			:species_attributes => [:code, :name, :id]]
		)
end

```

Due to the multifaceted layers of nesting, associations and validations, getting everything working involved some mind-bending chicken/egg convolutions in which I couldn't create the instance of Model A until I saved the instance of Model B but I couldn't save the instance of Model B until I created the instance of Model A. My memory is a little vague, but I believe I had to establish a portal with a parallel universe to get that to work. I also wound up writing a custom birds_of_species_attributes= method that's bristling with ifs and elses and instance variables and deeply nested hash elements and a liberal sprinkling of ugly helper methods. There's even a `return` in there. Please don't judge me.

## Validations

Speaking of validations, that was another significant hurdle to overcome. Because the form was tied to report, the only error messages that were returned were those associated with the Report model. However, my validations -- and therefore my meaningful error messages -- were associated with Species and BirdsOfSpecies. As a result, my validations intially returned extremely unhelpful error messages like: `Birds of species is invalid`, which is not only vague, but also refers to a model that isn't meaningful to the user. To implement meaningful errors, I had to create custom validation methods that check whether species and birds_of_species are valid and, if they aren't, copy the associated error messages (which are stored in the species and birds_of_species objects) into the report object: 

```
class Report < ApplicationRecord
    ... 
    validates :date, uniqueness: true
    validate :new_species_is_valid
    validate :new_bird_of_species_is_valid

    def new_species_is_valid
        if @new_species && !@new_species.valid?
            errors.add(:species, ": name #{@new_species.errors[:name].last}") if @new_species.errors[:name].last
            errors.add(:species, ": alpha code #{@new_species.errors[:code].last}") if @new_species.errors[:code].last
        end
    end

    def new_bird_of_species_is_valid
        if @new_bird_of_species && !@new_bird_of_species.valid?
            errors.add(:species, ": number banded #{@new_bird_of_species.errors[:number_banded].first}") if @new_bird_of_species.errors[:number_banded].first
            errors[:birds_of_species].clear
        end
    end

```

Furthermore, to get the validation errors for the deeply nested model (species) to show properly on the form, I also had to hand-code the trigger for the `field_with_errors` class. To do this, I changed the code for the two text fields (alpha code and species name) from:

```
<td><%= s.text_field :code %></td>
<td><%= s.text_field :name %></td>
```

to:

```
<td class="<%='field_with_errors' if code_error_exists(report) && last_record?(b, report)%>"><%=s.text_field :code%></td>
<td class="<%='field_with_errors' if name_error_exists(report) && last_record?(b, report)%>"><%=s.text_field :name%></td>
```

and added the following helper methods:

```
def code_error_exists(report)
		!report.errors[:species].empty? && report.errors[:species].any? { | str | str.include?("code") }
end

def name_error_exists(report)
		!report.errors[:species].empty? && report.errors[:species].any? { | str | str.include?("name") }
end

def last_record?(bird_of_species, report)
		bird_of_species.object == report.birds_of_species.last
end
```

This resulted in validation error messaging that looked like this:

![](http://burtonux.com/flatiron_blog/validation-errors.jpg)

Much better.

## One last thing

Why the last_record? method, you might ask. Well that was because I tried to get clever with mimicking functionality that would add records dynamically. When a user starts a new report, they are taken to the /reports/new form:

![](http://burtonux.com/flatiron_blog/data-entry-form.jpg)

Once a user enters the date and the first species and submits the form (assuming there aren't any validation errors), the edit page is then rendered with this code:

```
def edit
		@report.birds_of_species.build.build_species
end
```

This build creates an instance of both birds_of_species and species to wrap the respective fields_for around, which means a line containing a blank set of entry fields appears after the line containing the bird that was already entered: 

![](http://burtonux.com/flatiron_blog/with-one-bird-added.jpg)

Because of this, I needed to specify that the `field_with_errors` class should only be added to the last record in the table. Without that check, every time the inner fields_for loop is executed:

```
<%= b.fields_for :species do |s|%>
		<tr>
				<td class="<%='field_with_errors' if code_error_exists(report)%>"><%=s.text_field :code%></td>
				<td class="<%='field_with_errors' if name_error_exists(report %>"><%=s.text_field :name%></td>
				<td><%= b.number_field :number_banded %></td>

		</tr>
<%end%>

```

the `error_exists` methods would return true and the `field_with_errors` class would be added, resulting in this:

![](http://burtonux.com/flatiron_blog/dodo.jpg)

even though only the last row contains a validation error. 

The workaround for the dynamic fields had one additional side effect: the final set of blank fields rendered after all the birds have been entered throws a validation error when you try to post the report. To handle that case, I created an `all_fields_blank?` method that ends the processing of the form before the validations occur: 

```
def birds_of_species_attributes=(birds_of_species_attributes)
		if all_fields_blank?
				return
		else
		...
end

```

Finally, alert readers may have picked up the fact that my edit form isn't really an edit form -- it's an add-more form. Clearly this is not a permanent solution. I'll be glad when I can program it right using JavaScript. 
