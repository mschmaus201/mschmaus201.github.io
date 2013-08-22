---
layout: post
title: "Lets Set Up ActionMailer"
date: 2013-08-06 23:37
comments: true
categories: 

---
To begin setting up ActionMailer, first step is to run *rails g mailer gg_mailer* (I’m using 'gg_mailer' as the name for my mailer class in this walk-through) in the terminal, and then bundle install.  This will create a two files for you:  
&nbsp;&nbsp;&nbsp;&nbsp;1. *setup_mail.rb* in config/initializers directory  
&nbsp;&nbsp;&nbsp;&nbsp;2. *gg_mailer.rb* in app/mailers directory    
  
##*setup_mail.rb*  
The first file to begin working with is the *setup_mail.rb* file where you will connect your email account to your rails app.  I'll be using GMail for this example.  The first step to set up this file is to establish the delivery method, SMTP, or simple mail transfer protocol.  SMTP is a protocol for sending emails between servers, and a large majority or mail servers use this protocol.  Next is to set up the SMTP settings.  First are the address, port, and domain for the mail server; for GMail you can use smtp.gmail.com, port 587, and gmail.com, respectively.  Then you include your username and password.  Next, you must establish what type of authentication is required – plain, login, or cram_md5.  You can use plain for GMail.  Using login will encode your password using Base64, and I don’t fully understand the mechanism of cram_md5, but according to the [ActionMailer documentation](http://api.rubyonrails.org/classes/ActionMailer/Base.html), it “combines a Challenge/Response mechanism to exchange information and a cryptographic Message Digest 5 algorithm to hash important information.”  Finally, you have to set the enable_starttls_auto value, which accepts a boolean value; for GMail set it to true.  This detects if your mail server uses STARTTLS (only available if your server uses SMTP), which encrypts the connection to an SSL or TSL connection rather than just a plain text connection.  
This is what it should look like by the time you’re done with this step.  
	ActionMailer::Base.delivery_method = :smtp
	ActionMailer::Base.smtp_settings = {
		:address              => "smtp.gmail.com",
		:port                 => 587,
		:domain               => "gmail.com",
		:user_name            => "gitgallery",
		:password             => "you_wish",
		:authentication       => "plain",
		:enable_starttls_auto => true
	} 
Just make sure to include this file in *.gitignore*!  Otherwise you'll be exposing your password to the world...  
##*gg_mailer.rb*  
Next file you have to work with is *gg_mailer.rb* in the app/mailers directory.  Here, you can set your default address for emails to be sent to, and define a method for each type of email you’d like to be sent.  Below you’ll see a method which sends off a registration confirmation email when called.  The reason for passing user in as an argument and assigning it to the local variable *@user* is to allow it to be in scope when you create the email in the views/gg_mailer directory.  In here you can include an attachment with the following syntax:  
	attachments["rails.png"] = File.read("#{Rails.root}/public/images/rails.png")  
Finally, the last line is what actually sends off the email (make sure it’s the last line of code!):  
	mail(:to => @user.email, :subject => "Registration Confirmation")  
Here is an example of what the file should look like.  

	class GgMailer < ActionMailer::Base
		default from: "gitgallery@gmail.com"
		
		def registration_confirmation(user)
			@user = user
			# attachments["rails.png"] = File.read("#{Rails.root}/public/images/rails.png") #USE THIS IF YOU WANT TO INCLUDE ATTACHMENT
			mail(:to => @user.email, :subject => "Registration Confirmation")
		end
		
		def new_project(project, contributor)
			@project = project
			@contributor = contributor
			mail(:to => @contributor["email"], :subject => "A repo you contributed to was just made into a project on Git Gallery.")
		end
	end
The final step to get ActionMailer working is simply to call the method you just wrote wherever you’d like in your program!  For the example I gave, I’m calling it in the user#update controller/action for various reasons, but essentially this method will only trigger when a user signs in for the first time and confirms their account info!  
##development_mail_interceptor  
One additional step you can take is to include the development_mail_interceptor, using the gem “mail”.  The mail interceptor allows you to ensure that all outgoing emails from the designated account direct to an assigned email address, usually the same one that the messages are being sent from.  This is so that whenever you’re in development, testing your site/seeding your database/etc, you are not constantly sending emails to addresses that may be included in your testing environment.  
  
So, for the mail interceptor, you want to create the file development_mail_interceptor.rb in the lib director.  In it, you want to have a class of *DevelopmentMailInterceptor*, and the class method of *delivering_email(message)*.  You can set the email subject and to address by declaring *message.subject* and *message.to*, respectively.  For the subject, it is recommended to include in there the email address in which the message is supposed to be sent to, and the message.to address should be sent to where you are sending from.  
  
The final step for the mail interceptor is to require it in the *setup_mail.rb* file, and to call  
	ActionMailer::Base.register_interceptor(DevelopmentMailInterceptor) if Rails.env.development?  

This is what the file should look like by the time you're done including the interceptor:

	require 'development_mail_interceptor'
	
	ActionMailer::Base.delivery_method = :smtp
	ActionMailer::Base.smtp_settings = {
		:address              => "smtp.gmail.com",
		:port                 => 587,
		:domain               => "gmail.com",
		:user_name            => "gitgallery",
		:password             => "you_wish",
		:authentication       => "plain",
		:enable_starttls_auto => true
	}
	
	ActionMailer::Base.register_interceptor(DevelopmentMailInterceptor) if Rails.env.development?

##View Files  
Last but not least is to actually create the view file for the email itself.  These files should be saved in the views/gg_mailer directory, with the same name as the method that is being called.  For example, the email file for a registration confirmation email (the method triggered for this is registration_confirmation) will be  registration_confirmation.html.erb.  You should also create a text file for those users who prefer text emails over html emails.  
Best of luck setting up ActionMailer in your own Rails app! Resources I found incredibly helpful were the railscast as well as the documentation.  
  
  Until next time!