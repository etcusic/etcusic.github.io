---
layout: post
title:      "CLI Project - Data Flow & Class Structure"
date:       2020-09-11 15:32:23 +0000
permalink:  cli_project_-_data_flow_and_class_structure
---


For my project I began with a data scrape from NFL Network's homepage to get information from each team in order to structure classes that represent different levels of organization: League, Team, Player. Getting data all the way down to the player data was a bit tricky, especially if I was going to do it as efficiently as possible. 

First I started on the NFL homepage to get some basic information from each team, including each team's url (all 32 team's webpages follow the same HTML structure). The url would take me to each team's website where I could pull some basic data from the players on their roster. 

However, accessing all 33 pages (1 for NFL homepage and 32 for each team) was going to be a lot of processing upon initialization. I decided to access some basic information from each team in order to create 32 new Team instances, which I could then access on an as needed basis using the team's url (now an instance variable of the team object).

```Nokogiri::HTML(open(@url)).css("ul.d3-o-footer__panel-links").each do |tag|
   tag.css('li a').each do |a|
       if a.attribute('data-link_type').value == '_all-teams'
           hash = {}
           hash[:name] = a.attribute('data-link_name').value
           hash[:division] = tag.text.strip
           hash[:conference] = tag.text.strip.split(' ')[0]
           hash[:url] = a.attribute('href').value + "team/players-roster/"
           self.initialize_team(hash)
       end
   end
end
 
def initialize_team(hash)
   hash[:league] = self
   new_team = Team.new(hash)
   self.teams << new_team
end```

(don't mind the nested loops. It was necessary to get that specific data and not unwanted bits)

As you can see, I used the concept of mass assignment in order to instantiate all 32 teams. This way I'm only scraping one webpage upon initialization, but still providing a lot of data with which to instantiate each team object (including divsion and conference). The League instance can then use its team objects to display basic information within the application. 

The next level of the application is essentially choosing which team to look at more closely. In the case of this application, I was just interested in the roster information. Once the user selects which team to look at, I used a similar data flow by accessing the team's roster url, scraping it for basic info on each player (position, height, weight, etc.), and then using the mass assignment concept again to instantiate each player from the Player class.

At this point I've instantiated 1 League, 32 Teams, and roughly 75 players from just 2 pages (comes out to roughly 2400 Player instances if all 33 web pages are accessed). 

For this specific application the structure is a bit overkill as I don't really do anything with the Player instances, and the NFL is the only League instance I'm using. The point of structuring it this way is to design it to be able to grow by having the data flow from the top => down. Once the data flows downstream, it can be used in either direction => (ex: a Team instance has 1 League object as an instance variable and 75 Player objects within an instance variable array).


