---
layout: post
title: "method_missing: do you know where your methods are?"
date: 2013-08-22 09:29
comments: true
categories: 
---  

This final week at the Flatiron School we had someone come so we can partake in mock interviews.  What an experience.   This was the first time I had gone through a technical interview, but I learned a lot more coming out of it than I thought I would!  Best lesson was learning about *method_missing* in Ruby.  
###NoMethodError  
Naturally, when you call an undefined method on an object, we’re all used to receiving this error:  
	NoMethodError: undefined method “#{method_name}” for "#{receiving_object_name}":“#{receiving_object_class}”  

But what exactly happens between the method being called and this error being raised?  If the current class does not contain the instance method being called, it checks if its parent class does.  If that class does not, it moves on up to that class’ parent class, all the way up until it reaches the outermost class: *BasicObject*.  *BasicObject* is the parent class for all classes in Ruby.  For more info on *BasicObject*, check out the Ruby docs [here](http://www.ruby-doc.org/core-2.0/BasicObject.html).  If that class does not contain the instance method called (which it most likely won’t because, by default, it’s a blank class), it will check the Kernel module.   If nothing comes up in the Kernel module, it will call *method_missing*.  By default, this is what raises *NoMethodError*, but of course Ruby graciously provides you the ability to override *method_missing*!  

###*method_missing*  

In a way, *method_missing* is a safety net.  It provides you with a way to gracefully handle calling a method that doesn’t exist!  

The prompt for my mock interview can be found [here](https://gist.github.com/wengzilla/e83f221f64077b635047).  I was asked to import a YAML file, and make the following string of methods on data return the relevant value in the YAML file:  

	parse_object.product.first.sku  
This string of methods should be the equivalent of calling:  
	parse_object[“product”].first[“sku”]  
This is where method missing kicks in! Rather than having to define a method for every possible key in the parse_object hash, you can abstract it all to the *method_missing* method.  By passing in as an argument the key you want to retrieve you can set the method as follows:  
	def method_missing(key)  
		ShippingHash.new(@data[“#{key}”]  
	end  
where *ShippingHash* is the class and *@data* is the instance variable (a hash) initialized upon creation of a new *ShippingHash* object. Below is the final code for the parsing program:  
	require 'yaml'  
	
	file = File.open("file_path/shipping.yaml")  
	data = YAML::load(file)  
	
	class ShippingHash  
		def initialize(hash)  
			@data = hash  
		end  
		
		def method_missing(key)  
			ShippingHash.new(@data["#{key}"])  
		end  
		
		def first  
			ShippingHash.new(@data[0])  
		end  
	end  
	
	parse_object = ShippingHash.new(data)
	puts parse_object.product.first.sku.inspect  
Overwriting and using *method_missing*, which is considered metaprogramming, may seem like a great way to make your program more abstract, and use less code.  But it is naturally slow as it must first check the current class, and all parent classes for the method name called before running *method_missing*.  