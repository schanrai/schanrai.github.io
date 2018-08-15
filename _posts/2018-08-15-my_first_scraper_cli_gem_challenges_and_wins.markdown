---
layout: post
title:      "My first Scraper CLI Gem | Challenges and Wins"
date:       2018-08-15 10:43:21 +0000
permalink:  my_first_scraper_cli_gem_challenges_and_wins
---

I had always loved the design of the [Startup Grind website](http://www.startupgrind.com/) but the UX and formatting of the [events page ](https://www.startupgrind.com/events/) did not make for an efficient or fast  browse or information retrieval process. So I decided to use the CLI Gem project as an opportunity to build a scraper that would facilitate easy retrieval of information about future Startup Grind events around the world.

I began (as I do with all my projects), by documenting my business/project requirements and including  any assumptions I have about how the data will be retrieved or any challenges from the outset.

> User/Business Requirements:
> 
> 1a. User can select events by location by selecting an option from a list of locations
> 
> 1b. OR User can select event from list of all events
> 

> 2a. User can view future list of events in their city (sorted by date? sorted alphabetically? unique?) Data that is presented to them is all from the index page
> 
> 3a. User can drill down to event details page for specific event and pull back detailed synopsis info, plus venue and time, and speaker. (how to get to speaker? concatentate address?)
> 

> 2b. User can view all the events in their chosen location 
> 
> 3b. By selecting from the mini-list of events in their selected location
>  
> 3c. They can drill down to the events details page for their chosen event to pull back All details from index page AND detailed synopsis info, plus venue and time, and speaker from event details page
> 
> 4a. User will then be presented with the option to view list of all events, or search by location, or exit.
> 

The requirements called for two main user flows - one of which would be a 3 stage process to get to the selected event details through location, the other, a 2 stage process to get to the selected event details page via the complete list of all future events. This list changes dynamically as the Startup Grind page updates, so hard-coding anything was not an option.To visualize the flow better and to help me plan out the domain model, I sketched out a simple flow chart:


![](https://i.imgur.com/MFVeBMk.png?2)


After watching the Daily Deals [video](https://www.youtube.com/watch?v=lDExWIhYKI), I decided to keep my domain model very simple (Scraper > Events > Cli) taking an '[inside out' development approach](https://softwareengineering.stackexchange.com/questions/166409/tdd-outside-in-vs-inside-out) and began by building out my scraper class and methods first. Why did I choose this route? I had the least amount of assumptions about this component of the application in terms of interactions and collaborators. I also knew that there would be a lot of trial and error to get the datapoints/attributes I needed, so decided to get the heavy lifting done early. 

## Scraper Methods

To get started quickly I just created a single file outside of the gem structure in my sandbox and got to work with Nokogiri.

From the outset I knew I would need one method for the events homepage(or index page) to retrieve basic information on each event, and a second one for the whichever event details page was requested by the user. In the interests of optimizing my application's performance , it made sense to only parse the DOM for event details pages that were specifically selected by the user.

The first challenge presented itself on the index page in the form of multiple nested divs! It took much trial and error to isolate a single event container. Accessing the **date element** and **event details URL** nested within also took a lot of work...an effort that was only superseded by the herculean task of figuring out why all datapoints for each queried elements were returning back for the whole page in each node queried, as opposed to single instances in each iteration. After much headscratching I discovered why, I was applying the .css method on the whole page, not the iterator of each event container...DOH!!



Constructing the scrape method for the event details [page](https://www.startupgrind.com/events/details/startup-grind-johannesburg-presents-come-party-with-joburgs-vibrant-entrepreneur-community#/) challenged me even further, but I really gained a much deeper understanding of the DOM and HTML/CSS through this endeavour. My proudest moment was refactoring the method to pull out the element from the **Date and Time**  section at the bottom of the page which was encased in `br` tags only and had no particular identifying class...**that is when I really learned that nodes could be treated like arrays** -  I went from this piece of ugliness:


`doc.css(".container-inner")[0].text.split("2018")[1].to_s.strip!.delete!("\r\n\\")`

To this:

`doc.at(".container-inner").children[8].text.strip`

Sadly, I then discovered that the content creaters for the website were inconsistent in how they formatted the Date and Time  vis a vis this [example](https://www.startupgrind.com/events/details/startup-grind-birmingham-uk-presents-we-are-hosting-simon-washbrook-founder-of-popcorn-email#/).
***NOTE this has now been corrected by the content creators (08/15/18).***


In the context of this project I realized that I could not spend hours trying to figure out the logic to address this inconsistency and I opted to scrape the start time of the event from the agenda section.  This is the risk with user-generated content (and a steady stream of interns)! Working in my last role as a product manager for a price optimization tool I was especially aware of this, and my team and I knew that we would likely have to rewrite our scrapers as many times as our competitors changed their layout/format...such is #scraperlife! 

Lastly, I concatenated all the address fields into one string to make it more readable on a terminal screen.

```

  #will take in the argument of a details_link from Event.fetch_details_url(event_id)
  def self.scrape_details_page(url)
    scraped_details = {}
    doc = Nokogiri::HTML(open(url))
    scraped_details[:long_descrip] = doc.css(".event-description").text.strip
    scraped_details[:start_time] = doc.at(".container-inner .agenda-item strong").text
    scraped_details[:address] = doc.at(".container-inner//span[@itemprop = 'name']").children.text + ", " +
      doc.at(".container-inner//span[@itemprop = 'streetAddress']").children.text + ", " +
      doc.at(".container-inner//span[@itemprop = 'addressLocality']").children.text + ", " +
      doc.at(".container-inner//span[@itemprop = 'postalCode']").children.text
    scraped_details[:speakers] = doc.css(".event-speaker-list//h2[@itemprop = 'name']").map {|y| y.children.text.strip}
    scraped_details
    end
```


## Environments, Github and the IDE

Once I had the code nailed for the Scraper component , I decided to setup my project and stub out my gem. Despite the fact that I followed the instructions ad verbatim on Kenlyn's super helpful [video](https://instruction.learn.co/student/video_lectures#/316) for how to setup your CLI project, and I was fairly comfortable in my knowledge of Git and Versioning,  I spent close to 3 hours trying to search and retrieve my project folder that I had cloned back onto my local environment. 

I had to attend an office hours session to get my head around what was happening - luckily Kenlyn came to my rescue! By default, when you open your IDE without forking from a repo on the Learn platform, you are automatically navigated into the temporary folder on your local environment which will delete any files stored within when exiting the IDE.  

I had accounted for this before my problems appeared and  created and  pushed up my local files to my remote repo on Github once I was confident with the Gem and Environment structure (Avi's [video](https://www.youtube.com/watch?v=XBgZLm-sdl8) on Environments, Gems and Bundlers really helped!) . However, once I cloned the project back to my local environment, despite being able to navigate to the directory and master branch in my terminal screen, I could not find a way to view  the project filders and structure in tree-view on the IDE. This was horribly disconcerting as you need to be able to jump back and forth between your library files and save quickly.

I soon discovered why - I was thinking too much and had cloned my project back into the directory above the temporay folder (/home)  -this is absolutely inadvisable! Do everything in temporary and commit and push often. I soon learned that if I didn't I would get logged out of the IDE and lose my work. I believe this is a deliberate 'feature' of the Learn IDE -it certainly got me to commit and push often and really understand how the idiosyncrasies of the Learn IDE.

## Event Methods - search, find and retrieve

Writing the Event class and methods finally got me to a point where I could confidently build my own class finders and operators - a necessity to be able to query and present the data back in the format that I wanted. For instance, I knew that in order to present a list of locations from which the user could choose to then drill down to event in their chosen location, I would need to deploy a class finder method and most likely create a class variable to store the locations array for recall every time the user returned to the location menu.

I was intimidated at the thought of this, but if I wanted a nice, alphabetically sorted list of unique locations, I would have to put my big girl pants on and just do it:

*Event Class*
```
  def self.list_locations
    self.all.map do | event|
      event.location
    end.uniq.sort
  end
```

*Cli Class*
```
def display_event_locations
  i = 0
  d = Event.list_locations
  puts ""
  puts "************* Event Locations *************"
  d.each do|location|
  @@locations_array << location #you need this for fetch_by_location
  puts "#{i += 1}. #{location}"
  end
end
```

At this stage, as I was still not sure how I wanted to build my CLI, I decided to document any important dependencies on other methods, objects or classes within comments on the code. Naming the methods after the function or action they perform is a good practice as it keeps you accountable to maintaining single responsibility. If the dependency requires data or arguments in a certain format, it's important to document these too as you go along  because you may move methods into other classes as you make changes or solidify decisions about the design of your application.

Before I started building the Event class I had no idea how I would test out each method now that the training wheels had come off and there were no Rspec tests neatly mapped out to rely on. Building a rake console task per Avi's suggestion in this [video](https://www.youtube.com/watch?v=Y5X6NRQi0bU) really helped as I could test each method out as I went along.

This Rake task basically loads all your library and environment up when you instanciate a pry using the command `rake console `. It was invaluable as I could actually construct my methods line by line and see the results as I was building them.

```
require_relative './lib/sgrind.rb'


task :console do
  Pry.start
end
```

 I could not have done this project without this task - it totally helped me maintain a good workflow. If there is one thing you should do before starting this project, it's watching all the videos I mention in this post. They are gold and will literally save you hours of head-scratching.

 
## Command Line Interface - Methodology

I am weak on loops and boolean operations so constructing the CLI was the part that I was dreading the most. I knew that I could not just jump and start coding, so I decided to map out all the functions I would need on my flow chart by each user process:

![](https://i.imgur.com/Dte5Doe.jpg?1)

This process really helped me keep on track as I built out the CLI - it also kept me calm and allowed me to focus on writing code that was resuable and (hopefully!) DRY.

Now that I could focus on the code and not have to think about the structure or design - I was able to knock out the CLI in a few hours. I ran into one small hitch, where I felt that I needed to move a method over into the CLI class from the Event class in order to call it without making it a class method. The original method ( when it lived in the Event class looked like this:


```
#this method operates on an argument of a single event object instance

  def list_event_details_by_location(event)
    url = event.details_link       #the details_link is the URL of the event details page 
    self.add_details(Scraper.scrape_details_page(url))  #adds the new attributes from the  event details page
  end
```


In order to call it within the CLI class I initially thought I would have to turn the `list_event_details_by_location` method into a Class method in the Event class so I could call it from the CLI like so:


```
def view_event_details(result)  #result is the array of filtered events by location
  puts "Enter a number from the numbered list of events to view details of your selected event:"
  input = gets.chomp
  #if (input.to_i >= 1 && input.to_i <= result.size   #error correction - implement at end
  event = result[input.to_i - 1]
  event_instance = Event.list_event_details_by_location(event)
  display_event_details(event_instance)
end
```

But this felt wrong (and if memory serves me right, I don't think it worked anyway!) - the `list_event_details_by_location` method was designed to work on a single object instance, if I changed it into a Class method just so I could call it from the Cli class as above then Self within the line `self.add_details(Scraper.scrape_details_page(url)) ` would not be performed on the object instance, it would operate on the Event class itself....no bueno!!

Moving the  method into the CLI class didn't feel right either :


```
def view_event_details(result) 
  puts "Enter a number from the numbered list of events to view details of your selected event:"
  input = gets.chomp
  #if (input.to_i >= 1 && input.to_i <= result.size    #error correction - implement at end
  event = result[input.to_i - 1]
  event_instance = list_event_details_by_location(event)
  display_event_details(event_instance)
end

  #reformatted this and put it in cli class from event - but is that correct?
  #call this within cli on event instance chosen by user from location mini-menu/array from fetch_by_location method
  #pass the event instance in as an argument
  #output will be event instance with additional atttributes from details page
	
  def list_event_details_by_location(event)
    url = event.details_link
    event.add_details(Scraper.scrape_details_page(url))
  end
```


I realized I was still at sea when it came to Self within the context of object collaboration and I needed some guidance(rescuing). Enter my knight-in-shining-armour AKA David Kennell from Project Support. With David's help, I refactored the  `view_event_details`  in the CLI class and kept the `list_event_details_by_location` method within the Event class.

![](https://i.imgur.com/TxwUP5B.png?1)

The variable `'z' `  is the event instance under consideration. David helped me  realize that I could call the `list_event_details_by_location` method on an event object instance within a CLI method, because when you define a method within an Event class, it becomes part of the methods you can call on an object of that class from anywhere. To see this in action, just try `Object.methods` on any object for which you've defined custom methods.

I refactored further to this:

```
  if input.to_i >= 1 && input.to_i <= result.size
    z = result[input.to_i - 1]
    display_event_details(z.get_event_details)
  else
    puts "Invalid input, please choose a number from the list of events:"
  end
```

## Conclusion

At this stage, I have no idea whether I will be asked to refactor large parts of the code but what I do know is that I have a working application according to my specifications. I have to be honest here and admit that it's been a real struggle for me to date on this course until I started this project... half the battle  has been understanding the requirements of many of the labs and the tests. In this scenario, I fully understood the requirements (as they were my own) and the motivation to solve problems on my own was higher because I was on my own.

Sometimes, being out there on your own is the best thing you can do for your confidence, personal and professional development. 

You can take a look at my project code [here](https://github.com/schanrai/sgrind-cli-app) and watch my video walkthrough [here](https://drive.google.com/open?id=19YKwAqkunX5stQUZYO1KmrvS5aIrDcfy).

