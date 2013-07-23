---
layout: post
title: "Rails Scales"
date: 2013-07-09 00:51
comments: true
categories: 

---


##Rails Scales  

Although I just began learning about the magic and wonders of Ruby on Rails, I’ve heard for some time that Rails has issues with scalability once you reach a certain point.  But in reality, that’s all I’ve really ever heard – I’ve never been offered any details as to how or why this is the case.  

###Popular Sites on Rails  

Before getting into the details, there are a number of very popular sites that are built using Ruby on Rails, including: GitHub, Groupon, Hulu, YellowPages, Basecamp, Shopify, and Airbnb.  

#### Twitter  

Twitter was launched using Rails back in 2006.  Beginning in 2008, rumors of Twitter dropping Rails surfaced in the news, but this didn’t actually occur until 2011 – switching to Lucene, a “full-featured text search engine library written entirely in Java.”  This transition took place the week after the tsunami hit in Japan, which led to nearly a doubling in the usage of Twitter.  The switch cut latency to a third.  BUT, that doesn’t mean that the scalability issues were related to Rails!  

### The Problem with Rails  

So most of the issues that arise with Rails don’t directly have to do with the Rails framework itself, rather the interaction with the database.  For example, Rails encourages programmers to develop in a local environment in which you generally work with smaller data sizes, but once deployed, you may find that working with larger data sizes can introduce significant performance problems.  Additionally, because Rails separates you a good amount from SQL, many Rails developers get into this “SQL is bad” mindset, instead relying on ActiveRecord.  Of course, ActiveRecord makes so many things so easy, but it does have its own performance issues.  

Turns out that ActiveRecord is the main culprit in scalability issues within Rails, specifically relating to the N+1 query issues!  Just for review, here’s an example of what the N+1 query issue is:  
`Lets say you have 100 cars, each with four tires.  In order to query each tire position (lets say depending on where the tire is, it has a position number of 1-4) with ActiveRecord, you’d ultimately be querying all 100 cars, and then for each car, you query the tires.  That ultimately comes out to 101 queries – incredibly inefficient!  Ultimately, it is faster to issue one query with 100 results rather than issue 100 queries with one result.`  

###Avoiding N+1  

Luckily, this isn’t something you have to be stuck with.  ActiveRecord uses something called lazy loading, in which it only performs a certain task when necessary.   On the other side of the spectrum is eager, and over-eager loading.  For example (based on a post [here](http://stackoverflow.com/questions/1299374/what-is-eager-loading)):  

Imagine a page with rollover images like for menu items or navigation. There are three ways the image loading could work on this page:  
1. Load every single image required before you render the page (eager);  
2. Load only the displayed images on page load and load the others if/when they are required (lazy); and  
3. Load only the displayed images on page load. After the page has loaded preload the other images in the background in case you need them (over-eager). 

Using eager loading helps minimize the number of queries.  Ideally, the above example with cars and tires would only come out to two queries, no matter how many cars there were.  Here is how one would use eager loading:  

#####N+1 query  

<% @cars = Car.all(@cars).each do |car| %>  
<p><%= car.tire.position %> </p>  
<% end %>  

#####Eager loading  
<% @cars = Car.find(:all, :include=>[:tire] %>  
<% @cars.each do |car|%>  
<p><%= car.tire.position %></p>  
<% end %>  
As mentioned before, this will generate at most two queries, no matter how many rows you have in the posts table.  
###Detecting Performance Problems  
One of the most powerful plugins is the "query_reviewer" which automatically analyzes SQL code that ActiveRecord generates for potential problems. It rates a page's SQL usage into one of three categories (Ok, Warning, and Critical), attaches warnings to any queries in which one is needed, and displays an interactive summary to help you determine which queries are not absolutely necessary.  Here's a [link](https://github.com/nesquena/query_reviewer) to the repo for the gem.  
  
###Conclusion  

All in all, the scalability issues of Ruby on Rails can be overcome.  The framework allows you to spend less time building and more time planning the architecture and how to scale.  Don't let is scare you off from the elegance of the framework.
