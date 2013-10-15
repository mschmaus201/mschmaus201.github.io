---
layout: post
title: "Rails Time Zones"
date: 2013-10-15 10:15
comments: true
categories: 
--- 

Rails Time Zones I’ve recently been working on some freelance projects, one of which is building an online reservation system for a chain of soon to launch hair salons.  One of the deeper rabbit holes I ended up in was dealing with time zones.  What happens when the hair salons are located in different time zones?  What happens when users are in different time zones and make reservations?  What happens when a user is in one time zone and makes a reservation for a location in another time zone?  
  
  I barely realized I had any issue with time zones until I needed to list all remaining reservations for the day, using
	reservation.to_date == Time.now.to_date && reservation.time >= Time.now  
from the method called on a location:
	def todays_remaining_resevations  
		res_array = Array.new  
		self.reservations.each do |reservation|  
			res_array << "#{reservation.time.strftime("%I:%M %p")} (#{reservation.user.full_name})" if reservation.time.to_date == Time.zone.now.to_date && reservation.time > Time.zone.now  
		end  
		res_array.count != 0 ? res_array.sort! : "There are no remaining reservations for today"  
	end  
This was functioning properly, except that the reservations were being saved in a different time zone than Time.now was spitting out.  
  
  Rails does a lot to help, but there are still a number of things that are very confusing until you really dive deep into how it works.  
  
##Default Time Zone  
First thing you should know is that all DateTime objects are stored in the database as UTC (Coordinated Universal Time).  By configuring the default time zone, Rails will use ActiveRecord to convert the data to the time zone you’ve designated.  To set the default time zone, in the config/application.rb file, set config.time_zone to the name of the time zone you’d like to be default.  
	config.time_zone = 'Eastern Time (US & Canada)'  
To get a list of all the time zone names,  run:  
	rake time:zones:all  
in terminal.  Now, whenever something is saved, it will automatically be saved in UTC, but will be converted to UTC from your designated default time zone.  And since I had set the default time zone, I then changed the above line of code to:  
	reservation.to_date == Time.zone.now.to_date && reservation.time >= Time.zone.now  
##Multiple User Time Zones  
So what happens if users make reservations from different time zones?  The user model should have an attribute for time zone (string).  That way, you can easily call *current_user.time_zone* when trying to determine what time zone a user is making a reservation from.  And with the knowledge of what time zone a location is in, you can easily make the appropriate adjustments!
##Rails Code  
So the way DateTime objects are created through a rails form, is that for each part of a date (ie month, day, year, hour, minute, etc), its given its own key in params.  For example, when a location is created an hours set in this project, *params["location"]["monday_open(1i)"]* equals the month value, *params["location"]["monday_open(2i)"]* equals the day value, *params["location"]["monday_open(3i)"]* equals the year value, and so on.  Then, those pairs will instantiate *params["location"]["monday_open"]* and passed into *Date.new(parameter)*.  Its actually pretty cool checking out the Rails code and see how this is done at the granular level!  Here's a [link](https://github.com/rails/rails/blob/d90b4e2/activerecord/lib/active_record/base.rb#L1811) to the relevant code :)
####Other great resources  
[Working with Time Zones in Ruby on Rails](http://www.elabs.se/blog/36-working-with-time-zones-in-ruby-on-rails)  
[About UTC](http://www.timeanddate.com/time/aboututc.html)  
[RailsCast #106: Time Zones (revised)](http://railscasts.com/episodes/106-time-zones-revised)