---
layout: post
title: "Building a Ruby CLI Gem"
date:   2016-02-28
categories: tutorial
---

Today I'm going to go over building a simple command line interface gem with Ruby. The gem will be called Git-Trend and will allow the user to see what projects are trending on Github in a particular language. Also the user will be able to quickly view the README file of a specific project to learn more.

To start we will generate a boilerplate directory of files that will serve as a good starting point for building out our features. Bundler makes this extremely easy. To generate the boilerplate directories bundler has a command `bundle gem <gem_name>` where `gem_name` is what you want to call your gem. In your terminal type `bundle gem git_trend`. Lets take a look at what bundler generates:

    ▾ bin/
        console*
        setup*
    ▾ lib/
      ▾ git_trend/
          version.rb
        git_trend.rb
    ▾ spec/
        git_trend_spec.rb
        spec_helper.rb
      CODE_OF_CONDUCT.md
      Gemfile
      git_trend.gemspec
      LICENSE.txt
      Rakefile
      README.md

Bundler gives us a great starting point for building out our gem. `bundle gem git_trend` generates a bin, lib and spec directory. The bin directory contains a console and setup executable. Lets go into the console executable.

    #!/usr/bin/env ruby

    require "bundler/setup"
    require "git_trend"

    # You can add fixtures and/or initialization code here to make experimenting
    # with your gem easier. You can also use a different console, if you like.

    # (If you use this, don't forget to add pry to your Gemfile!)
    # require "pry"
    # Pry.start

    require "irb"
    IRB.start

This is gives us an irb console to experiment with our gem code. The console defaults to IRB, however, as you can see from the commented out code you can also use Pry. Lets swich our console to Pry. Comment our the pry code and comment in the irb code. We also need to add pry to our gemfile.

If you open up `Gemfile` you'll a comment instructing us to specify any gem dependencies in our gemspec file. Lets open up `git_trend.gemspec` and add pry.

`spec.add_development_dependency "pry"`

While were in `git_trend.gemspec` lets add a few more gems we will be using later on and run `bundle install`.

`spec.add_development_dependency "nokogiri"`

`spec.add_development_dependency "colorize"`

Before we build our features lets create an executable for our gem called gittrend. Type `touch bin/gittrend` and add the following code.

    #!/usr/bin/env ruby

    require "bundler/setup"
    require "git_trend"

    GitTrend::CLI.new.call


Ok now we can get down to building out our features. We will be scraping the data containing trending projects directly from Github. To scrape the data we will be using Ruby's built in open-uri and Nokogiri, which we installed earlier. Lets take a moment to list our gem objectives.

1. Greet the user
2. Ask the user what language they would like to see trending
3. Display a numbered list of trending projects specific to that language
4. Ask the user if they would like to read a README of any specific project
5. Ask the user for a project number
6. Display the project README
7. Loop back to ask user what language they would like to see trending
8. Exit program

To accomplish these objectives we can encapsulate all of our logic within two classes, the Scraper and CLI class. Lets build out the Scraper class first.

{% highlight ruby linenos %}
require 'nokogiri'
require 'open-uri'
require 'pry'
require 'colorize'

class GitTrend::Scraper

  def self.get_page(lang)
    doc = Nokogiri::HTML(open("https://github.com/trending/#{lang}"))
    projects = []
    doc.css(".repo-list-item").each do |project|
      title = project.css(".repo-list-name a").attribute("href").value.split("/")[-1].capitalize.green
      description = project.css(".repo-list-description").text.strip
      readme = "https://github.com" + project.css(".repo-list-name a").attribute("href").value
      projects << {title: title, description: description, readme: readme}
    end
    projects
  end

  def self.get_readme(project)
    doc = Nokogiri::HTML(open("#{project}"))
    readme = doc.css("#readme").text.strip.sub("README.md", "")
    puts readme
  end

end
{% endhighlight %}

First we require our dependencies and define our class. The Scraper class contains two class methods for extracting our data from Github, `.get_page` and `.get_readme`. Each method takes one argument which will be providing by the user. Lets examine the `.get_page` method. Once the user has provided a language to search, we assign the variable `doc` to the parsed JSON output provided by Nokogiri. Next we assign `projects` to an empty array. We will store each projects title, description and readme in a hash and push each hash into our array. Nokogiri allows the output stored in `doc` to be parsed further based on css elements. The Github trending page encapsulates each project in the class `repo-list-item`, so we can easily gather all list items and iterate through the array. Within the each block we then extract the title, description and readme of each project. Note that `#green` method adds a nice green color to the string, thanks to the colorize gem! Finally we push the individual project hash into the projects array and return the projects array.

The second class variable, `.get_readme` takes a url. The doc variable is then assigned to the Nokogiri output using the interpolated project argument. Finally we extract the README section we want and `puts` to the screen.  Now that we have our Scraper class done we can implement our CLI class.

The CLI class will be in charge of controlling the flow of our program and collecting and displaying data from the Scraper class. Lets take a look.

{% highlight ruby linenos %}

class GitTrend::CLI

  def call
    greet_user
    set_lang
  end

  def greet_user
    puts "Welcome, lets see what's trending in your language."
  end

  def goodbye_user
    puts "Stay Trendy!"
    exit
  end

  def request_info
    puts "Would you like to see a README of a trending project?"
    input = gets.strip.downcase
    if input == "yes"
      true
    else
      false
    end
  end

  def display_readme(lang)
    puts "Choose number for project"
    input = gets.strip.to_i - 1
    project = GitTrend::Scraper.get_page(lang)
    readme = project[input][:readme]
    GitTrend::Scraper.get_readme(readme)
  end

  def set_lang
    puts "What language would you like to search?"
    input = gets.strip.downcase
    while input != "exit"
      display_projects(input)
      if request_info
        display_readme(input)
      else
        set_lang
      end
    end
    goodbye_user
  end

  def display_projects(lang)
    projects = GitTrend::Scraper.get_page(lang)
    projects.each_with_index {|e, i| puts "#{i+1}. #{e[:title]} -- #{e[:description]}" }
  end
end
{% endhighlight %}

The CLI class controls the flow of our gem and utilizes the class methods we defined in our Scraper class to parse the data. With this class in place we can go to our terminal and type `bin/gittrend` to run our gem. Congratulations you've built a Ruby gem!
