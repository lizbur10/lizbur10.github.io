---
layout: post
title:      " Three Things to be Thankful For"
date:       2018-01-06 21:58:12 +0000
permalink:  three_things_to_be_thankful_for
---


I had my review for the CLI Data project a few days ago. Something happened in that review that I was expecting to happen: I lost my marbles. But I didn't remotely anticipate the sheer extent to which my brain -- which, after all, had been spending quite a bit of time learning this stuff in recent weeks -- lost all ability to reason when confronted with a request to refactor code. I was, in that moment at least, utterly incapable of thinking through how to solve the problem. 

Three things saved me.

First (and foremost) was my instructor. She very patiently talked me through what we were trying to do: for some reason, my program -- which was scraping information about events currently going on in Boston -- had code in the Scraper class that was storing information about event categories. The Scraper class, the instructor gently reminded me, shouldn't really be responsible for storing information about event categories, right? Um, yeah, right, I said. So what *should* be responsible? she asked. Well, I said, the Category class I guess. (Learning was happening, slowly and painfully.) OK, she said, so how do we go about doing that?  Well, hm, I don't really know, but I suppose instead of scraping the category information and storing the information in a hash which is then `puts`ed out by the CLI so the user can pick a category which then a) launches the scraping of the events belonging to that category, and b) instantiates the Category object, I could just instantiate the Category objects at the time I initially scrape the categories. Brilliant! she exclaimed (or words to that effect). I still had to actually do it, of course, but now I was faced with a task that I actually remembered how to do. Which brings me to the second thing: muscle memory.

I'd instantiated a lot of objects as I made my way through OO Ruby. Typing ClassName.new was pretty much automatic at that point. Of course, I first had to remember what I'd named the class (oh, wait -- we just said it -- Category!) as well as the fact that it was name-spaced and what the name-spacing was, but I was off and running. Once I instantiated the object, it was the easiest thing possible to access the attributes of the category objects to `puts` a numbered list of categories because, that too, I'd done a whole bunch of times recently. But I wasn't out of the woods yet: errors were happening. Which, as it happens, is the third thing.

We have all heard more than once that errors are good, but going through the review gave me a much greater appreciation for just how magnificent they really are. Most importantly, of course, they give you big juicy hints about what's wrong with your code. But almost as importantly, instead of being faced with a sea of broken code on a host of pages, you're faced with one tiny little error. You know exactly what you need to do next: fix it! Then, once *that* error is fixed, hey look at that! Another error to fix! The errors felt like old friends: I'd seen them all many times before so, in most cases, I knew pretty quickly what I needed to do. By this point, frankly, I was having fun. I've always loved puzzles (not surprising, I suppose, for an aspiring developer) and I was knocking out those suckers like ducks in a shooting gallery!

In the end I learned several lessons about how to survive a review (or technical interview):

1. Do lots of lessons and lots of labs and lots of reviewing of lessons and labs before you get there. And code code code. Your brain will come through for you even if you lose all control over it.
2. Know your code. Even if you forget everything in the stress of the moment, as soon as you start interacting with it you'll remember enough to keep you moving forward.
3. Talk things out. If you can't immediately answer the question, just start with what you do know; talking it through (to a rubber duckie, a pet, or a Flatiron School instructor) will help get you to the answer.

I can't pretend that I sailed through my review with no difficulty. I most definitely lost my marbles, and there were a few cringe-worthy moments I wish I could put out of my mind forever (like when I forgot the word for constant). But, as it turns out, I had the preparation to get through it, and that felt pretty great. 
