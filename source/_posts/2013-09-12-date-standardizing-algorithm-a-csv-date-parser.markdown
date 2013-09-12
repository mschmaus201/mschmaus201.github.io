---
layout: post
title: "Date Standardizing Algorithm: a CSV Date Parser"
date: 2013-09-12 14:45
comments: true
categories: 
---  

So this interview process has had its up and downs, having graduated from the Flatiron School about three weeks ago.  And those ups can sway to downs within moments.  But the highlights have definitely been solving the problems that are thrown my way, notably parsing differently formatted dates in CSV’s and creating *DateTime* objects out of them.  This was the prompt emailed to me by the interviewer:  

## The Prompt
*We allow customers to upload files (CSV, or otherwise delimited) directly to COMPANY REDACTED through a batch process.  During initial setup, we ask our customers to define the structure of these files.  That tends to work pretty well, save for one aspect: date formats.  The problem is, we can't rely on Ruby's date parser (it's too slow, and more importantly it can often be wrong when looking at individual dates), so we need to know the date format up front.*  

*Customers tend to not fill out the date format (and leave the default), or just select the wrong format altogether.  For the purposes of this exercise, let's assume that no UI solution is possible (we just want to look at the string, and figure out the format).  
I've attached 8 sample date files with varying date formats.  What we want to do is for each file figure out the format of the date field (you can take the column as an input argument -- no need to guess which column has dates).  We should then be able to supply the value in each row to Ruby's Date strptime method, along with the format we computed, and get back a Date object with the correct date.  Ideally, we'd also figure out the time format if it exists (and thus use with DateTime instead), but we don't necessarily need to start there.*  

*You'll notice all kinds of different varieties of dates.  Some have no separators between tokens (e.g. November 11 2012 might be 20121111), others have periods separating things.  Month / year / day can be in all kinds of different places and can take many shapes, e.g. with or without leading 0s.*  

*In many cases you'll need to scan multiple records before you can find a definitive answer, but we should be focused on being performant -- meaning, it's ok to parse many records if needed, but we want to do this in real time so we want to read as few records as possible to find the format conclusive, and then stop reading from the file.*  

*We can make obvious assumptions to make things easier (e.g., since e-commerce didn't exist in 1922, if you encounter a 22 you can assume it's a day), and we can assume years are either 2 digits or 4 digits (since there is no date format string for a single digit year).*  

## The Code
Before walking you through what I wrote, I figured it would be most logical to show you the final product, and then explain the steps the program takes to parse the dates, standardize the format, and create DateTime objects out of them.  Here’s the code ([also found at this repo](https://github.com/mschmaus201/csv_date_parser)):  
	require 'csv'
	require 'debugger'

	class CSVParser

	  MONTHS_ARRAY = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]

	  def initialize(file_path, column)
	    @file = file_path
	    @column = column
	    @date_column = get_date_column
	  end

	###############
	# MAIN METHODS #
	###############
	  def parse_date_format
	    date_indexes = learn_date_format
    	learned_date_indexes = fill_in_last_value(date_indexes)
    	datetime_creation(learned_date_indexes)
      end
      
      def format #RETURNS THE FORMAT OF THE COLUMN'S DATES
	    date_indexes = learn_date_format
	    learned_date_indexes = fill_in_last_value(date_indexes)
    
	    format = Array.new
	    format[learned_date_indexes[:year]] = "YYYY"
	    format[learned_date_indexes[:month]] = "MM"
	    format[learned_date_indexes[:day]] = "DD"
	    date_format = "#{format[0]}/#{format[1]}/#{format[2]}"
	  end

	private
	#########################
	# RUN ON INITIALIZATION #
	#########################
	  def get_date_column
	    date_column = []
	    CSV.foreach(@file) do |row|
	      date_column << row[@column - 1]
	    end
	    standardize_format(date_column)
	  end

	  def standardize_format(date_column) #DOESN'T WORK FOR DATE1.CSV
	    date_column.each do |date|
	      #FORMATS PUNCTUATION CORRECTLY
	      date.gsub!(",", "")
	      date.gsub!("-", "")
	      date.gsub!(".", "/")

	      #TRANSLATES SPELLED OUT MONTHS INTO REPRESENTATIVE INTEGERS
	      MONTHS_ARRAY.each_with_index do |month, index|
	        date.gsub!(/(#{month})[a-z]{0,}/i, " #{index + 1} ")
	      end

	      #REMOVES TIME INFO
	      date.gsub!(/\d{1,}:\d{0,}((AM)|(PM))/i, "")
	      date.gsub!(/\d{1,}:\d{0,}/, "")
	      date.gsub!(/\d{1,}((AM)|(PM))/i, "")

	      #ADDS/REMOVES NECESSARY/UNNECESSARY /'S
	      date.gsub!(" ", "/")
	      date.gsub!(/\/{1,}$/, "")
	      date.gsub!(/^\/{1,}/, "")
	      date.gsub!(/\/{1,}/, "/")
	    end
	    date_column.shift if date_column.first != /\d{1,}(\/)\d{1,}(\/)\d{1,}$/ #REMOVES TOP ROW IN DATE IF ITS A HEADER
	    date_column
	  end

	######################
	# SUPPORTING METHODS #
	######################
	  def learn_date_format
	  #ITERATES THROUGH EACH ROW AND LEARNS THE DATE FORMAT
	    #IE: MM/DD/YYYY, DD/MM/YYYY, YYYY/MM/DD, ETC
	    date_indexes = Hash.new
	    @date_column.each do |date|
	      split_date = date.split("/")
	      split_date.each_with_index do |x, i|
	        # study_format_learning(split_date, date_indexes) #UNCOMMENT ONLY TO STUDY ITERATION AND DATE FORMAT LEARNING
	        next if i == date_indexes[:day] || i == date_indexes[:month] || i == date_indexes[:year]

	        if year_indexed?(date_indexes)
	          date_indexes[:day] = i and break if x.to_i > 12
	        end
	        if month_indexed?(date_indexes)
	          date_indexes[:year] = i and break if x.to_i > 31
	        end
	        if day_indexed?(date_indexes)
	          date_indexes[:year] = i and break if x.to_i > 12
	          date_indexes[:month] = i and break if x.size == 1
	        end
	        if nothing_indexed?(date_indexes)
	          date_indexes[:year] = i and break if x.size == 4 || x.to_i > 31
	          date_indexes[:day] = i and break if x.to_i > Time.now.year%100 && x.to_i <= 31
	        end
	      end
	      break if two_indexed?(date_indexes) #STOPS ITERATION ONCE FORMAT IS LEARNED
	    end
	    date_indexes
	  end

	  def nothing_indexed?(date_indexes)
	    !year_indexed?(date_indexes) && !month_indexed?(date_indexes) && !day_indexed?(date_indexes)
	  end

	  def two_indexed?(date_indexes)
	    year_and_month_indexed?(date_indexes) || month_and_day_indexed?(date_indexes) || day_and_year_indexed?(date_indexes)
	  end

	  def year_indexed?(date_indexes)
	    date_indexes[:year] ? true : false
	  end

	  def month_indexed?(date_indexes)
	    date_indexes[:month] ? true : false
	  end

	  def day_indexed?(date_indexes)
	    date_indexes[:day] ? true : false
	  end

	  def year_and_month_indexed?(date_indexes)
	    year_indexed?(date_indexes) && month_indexed?(date_indexes)
	  end

	  def day_and_year_indexed?(date_indexes)
	    day_indexed?(date_indexes) && year_indexed?(date_indexes)
	  end

	  def month_and_day_indexed?(date_indexes)
	    month_indexed?(date_indexes) && day_indexed?(date_indexes)
	  end

	  def fill_in_last_value(date_indexes)
	  #FIGURES OUT WHICH VALUE HAS NOT BEEN DETERMINED AND ASSIGNS IT THE REMAINING INDEX
	    learned_date_indexes = date_indexes
	    indexes = [0, 1, 2]

	    #DELETES INDEXES WHICH HAVE ALREADY BEEN ASSIGNED
	    indexes.delete(learned_date_indexes[:day]) if learned_date_indexes[:day]
	    indexes.delete(learned_date_indexes[:month]) if learned_date_indexes[:month]
	    indexes.delete(learned_date_indexes[:year]) if learned_date_indexes[:year]
    
	    #ASSIGNS REMAINING INDEX TO VALUE THAT HASN'T BEEN DETERMINED
	    learned_date_indexes[:year] = indexes.first if !learned_date_indexes[:year]
	    learned_date_indexes[:month] = indexes.first if !learned_date_indexes[:month]
	    learned_date_indexes[:day] = indexes.first if !learned_date_indexes[:day]

	    learned_date_indexes
	  end

	  def datetime_creation(date_indexes)
	    @date_column.collect do |date|
	      if date.split("/")[date_indexes[:year]].size == 2
	        DateTime.strptime("20#{date.split("/")[date_indexes[:year]]}-#{date.split("/")[date_indexes[:month]]}-#{date.split("/")[date_indexes[:day]]}", "%Y-%m-%d")
	      else
	        DateTime.strptime("#{date.split("/")[date_indexes[:year]]}-#{date.split("/")[date_indexes[:month]]}-#{date.split("/")[date_indexes[:day]]}", "%Y-%m-%d")
	      end
	    end
	  end

	#######################
	# DEVELOPMENT METHODS #
	#######################
	  def study_format_learning(split_date, date_indexes)
	    puts "#{split_date} day:(#{date_indexes[:day]}) month:(#{date_indexes[:month]}) year:(#{date_indexes[:year]})"
	  end
	end

	csvparser = CSVParser.new("#{Dir.pwd}/csvs/date7.csv", 3)
	puts csvparser.parse_date_format
	puts csvparser.format

## The Walkthrough
For the purpose of this exercise, you’ll be dealing with one CSV at a time, each of which have a column of dates formatted in a certain way.  Each CSV will have dates formatted in a different manner, but a column of dates within a given CSV will have one uniform format.  When you create an new object of the *CSVParser* class, you must include the file path and name, as well as the column number which contains the dates.  An instantiation of a *CSVParser* object should look like this:  

	csvparser = CSVParser.new("#{Dir.pwd}/csvs/date7.csv", 3)  

The initialize method takes in the file/file path and assigns that to the instance variable of *@file*, and the column number is assigned to *@column*.  Using my *get_date_column* and *standardize_format* methods, the *@date_column* instance variable is assigned an array of dates formatted in the format, "xx/xx/xx".   The *get_date_column* simply feeds the *standardize_format* method an array of dates from the column number identified.  *standardize_format* then uses regular expressions to shape the dates, written in nearly any logical format (emphasis on logical), into the "xx/xx/xx" format.  

Given the year, month, and day values can be written in many different ways (ie with or without leading zeros, months can be spelled out or using representative numbers, etc) and orders (ie “06 Mar 2013” vs. “Mar 06 2013” vs. “2013 Mar 06”, etc), the tricky part is figuring out which order the format is in (MM/DD/YYYY vs. YYYY/MM/DD, etc).  That’s where the *learn_date_format*, and the gist of the program come in.  

For this method, you take one date string at a time from the *@date_column*, split it on “/”, and iterate through each index.  The algorithm, whose logic follows, determines which order this date is written in.  My logic for this algorithm was:  

1) Nothing has been indexed yet  
If the size of the element is 4, or greater than 31, the element has to be a year  
If the element is greater than the last two digits of the current year and is also less than or equal to 31, the element has to be a day  
2) The year is indexed  
If the element is greater than 12, it has to be a day  
3) The month is indexed  
If the element is greater than 31, it has to be a year  
4) The day is indexed  
If the element is greater than 12, it has to be a year  
If the size of the element is 1, it has to be a month  

As soon as one of these conditionals is true, it assigns the index of the current element to a hash, with the key of what value it represents.  For example, if nothing is indexed and it comes across an element who’s size is 4, *date_indexes[:year]* is set equal to the index of that element.  Then, as the method continues to iterate through more dates, it will skip that index, knowing that all values in that index represent years.  

Additionally, once a second value is indexed, you logically know what the third value is.  As soon as this is true, the method breaks out of the iteration.  There’s no point in continuing to iterate through the dates once the format is discovered.  It then fills in the remaining value with the unassigned index, using the *fill_in_last_value* method.  

The final step is to create *DateTime* objects out of the *@date_column* array, using the discovered format.  Within the *datetime_creation* method, it will prepend the year with “20” if a year is written as “13” rather than “2013”.  

All methods except for the *parse_date_format* and *format* methods are private (the *initialize* method is private by default).  The *parse_date_format* returns an array of *DateTime* objects, and the *format* method returns the format of the standardized date as a string (ie “MM/DD/YYYY”).  Lastly, I also included a method, *study_format_learning*, which is commented out.  But once you comment this in (does that make sense?), you’ll be able to see the iteration through dates, watching it break (as in break out of the iteration) as soon as two of the values are indexed.  This is just to ensure that unnecessary iterations are not taking place.  

##Improvements
There is, and always will be, continued improvement to be made.  The first step for someone to take to improve on this would be to beef up the *standardize_format* method’s regular expressions to account for a larger variety of dates.  Additionally, the next step would be to account for times as well, as in its current state, this program only creates dates.  

Again, [here's the repo](https://github.com/mschmaus201/csv_date_parser) to take a look at the code more thoroughly!