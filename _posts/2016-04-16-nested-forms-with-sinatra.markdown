---
layout: post
title: "Building Nested Forms with Sinatra"
date:   2016-04-16
categories: tutorial
---

Nested forms are a great way to allow users to input interconnected data all within one form. In this tutorial we are going to build a simple survey form that will allow a user to persist data with three model objects: Survey, Question, and Answer. First lets get our models set up.

`survey.rb`

{% highlight ruby linenos %}
class Survey
  attr_reader :name

  SURVEYS = []

  def initialize(args)
    @name = args[:name]
    SURVEYS << self
  end

  def self.all
    SURVEYS
  end
end
{% endhighlight %}

`question.rb`

{% highlight ruby linenos %}
class Question
  attr_reader :content

  QUESTIONS = []

  def initialize(args)
    @content = args[:content]
    QUESTIONS << self
  end

  def self.all
    QUESTIONS
  end
end
{% endhighlight %}

`answer.rb`

{% highlight ruby linenos %}
class Answer
  attr_reader :content

  ANSWERS = []

  def initialize(args)
    @content = args[:content]
    ANSWERS << self
  end

  def self.all
    ANSWERS
  end
end
{% endhighlight %}

Within each model we are persisting a new instance at initialization. Now we can write our `app.rb` file to set up the routes we'll need for the form post our data and redirect up to a nice show page.

{% highlight ruby linenos %}
class BlogNestedForms < Sinatra::Base

  set :public_folder => "public", :static => true

  get "/" do
    erb :welcome
  end

  get "/new" do
    erb :"survey/new"
  end

  post "/survey" do
    @survey = Survey.new(params[:survey])

    params[:survey][:questions].each do |question|
      Question.new(question)
      Answer.new(question[:answers])
    end
    @questions = Question.all
    @answers = Answer.all


    erb :"survey/show"
  end
end
{% endhighlight %}

Now we can get to the meat of nested forms, the form! Since we have multiple questions within a single survey, we need a way to organize the questions. Lets add a dash of ERB magic to pass the question data into an indexed array. Using `[]` allows ERB to automagically index the questions hash.

{% highlight html linenos %}
<form action="/survey" method="post">

  <h1>Survey</h1>
  <label for="survey_name">Survey Name</label>
  <input type="text" name="survey[name]">

  <h2>Questions</h2>
  <label for="question_content_1">Question 1</label>
  <input type="text" name="survey[questions][][content]"><br>
  <label for="answer_content_1">Answer</label>
  <input type="text" name="survey[questions][][answers][content]"><br>

  <label for="question_content_2">Question 2</label>
  <input type="text" name="survey[questions][][content]"><br>
  <label for="answer_content_2">Answer</label>
  <input type="text" name="survey[questions][][answers][content]"><br>

  <label for="question_content_3">Question 3</label>
  <input type="text" name="survey[questions][][content]"><br>
  <label for="answer_content_3">Answer</label>
  <input type="text" name="survey[questions][][answers][content]"><br>

  <input type="submit" value="Submit">

</form>
{% endhighlight %}
If we submit this form we get back the following params hash:

{% highlight bash linenos %}
"params" => {"survey"=>
{"name"=>"Dog Survey",
"questions"=>
  [{"content"=>"What is your favorite dog?", "answers"=>{"content"=>"Lab"}},
  {"content"=>"What is your dogs favorite hike?", "answers"=>{"content"=>"Point Reyes"}},
  {"content"=>"What is your dogs favorite food?", "answers"=>{"content"=>"Salmon"}}]}}
{% endhighlight %}


As you can see in our `app.rb` file the `post "/survey"` action uses this params hash to iterate through each question and persist the survey, questions and answers.  With our instance variables we can create a nice show page to list out the survey name and the question and answer content.

{% highlight erb linenos %}
<h1><%= @survey.name %></h1>
<% @questions.zip(@answers) do |q, a| %>
  <h2><%= q.content %></h2>
  <h3><%= a.content %></h3>
<% end %>
{% endhighlight %}

The `#zip` method comes in handy here to alternate between the two arrays `@questions` and `@answers`. This is the basic overview of nested forms in Sinatra. In a later post I will go over how to code the same nested form with Rails and ActiveRecord.
