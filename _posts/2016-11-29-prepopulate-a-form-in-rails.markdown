---
layout: post
title: "Prepopulate a Form in Rails"
date:   2016-11-29
categories: tutorial
---

Recently I was working on a Rails CRUD application that allowed users to organize camping trips. The application also retrieved campsite data for different national parks via the Active API. The basic user flow of the application consisted of 
1) The user logs into application and is redirected to user show page.
2) The user show page contains a search form to search for national parks by name.
3) If the national park name is valid, the search query is used to search the Active API and return a XML response of campsite data unique to that national park.
4) Once this data has been retrieved the user is redirected to a results page that neatly displays the campsite names.

Once the user has a list of campsites specific to the national park they want to visit I wanted to be able to create a new trip based on the campsite name. Easy, I just add a link to create a new trip for each row in the table. Now when the user clicks Create Trip they will be redirected to a new trip form.

![blank form](/assets/blank_form.png)

Oh no, the form is blank! This user flow is making the user remember what campsite they clicked on and enter the name into the form again. This is a horrible experience from the users point of view. What I really want is for the form to be prepopulated with the name data from the row the user selected.  Luckily, this is super easy in Rails. Lets do it!

First, let looks at our `results.html.erb` file.

{% highlight erb linenos %}
<h1>Here are your results for <%= @json["resultset"]["pname"].titlecase %></h1>

<table class="table">
    <% @json["resultset"]["result"].each do |park| %>
      <tr>
        <td>
          <%= park["facilityName"].titlecase %></br>
          <button><%= link_to "Create Trip", new_trip_path %></button>
        </td>
      </tr>
    <% end %>
</table>
{% endhighlight %}

Here we have the table and for each row we have a corresponding button that links to the new trip form. We need to change the parameters passed to the new_trip_path method slightly to assign the name attribute to the row data.

{% highlight erb linenos %}
<td>
  <%= park["facilityName"].titlecase %></br>
  <button><%= link_to "Create Trip", new_trip_path(name: "#{park["facilityName"].titlecase}") %></button>
</td>
{% endhighlight %}

Second, we need to modify our `trips_controller.rb` file to handle this name attribute that is being passed.

{% highlight ruby linenos %}
class TripsController < ApplicationController
  def new
    @trip = Trip.new
    @name = params[:name]
  end
end
{% endhighlight %}

Here we define a new action and set the instance variable @name to params[:name] that is getting passed from out button link.

The final step is to add a value and set it to our instance variable @name to our new form.

{% highlight erb linenos %}
<h2>Lets get ready for your trip to <%= @name %></h2>
<%= form_for @trip do |f| %>
  <%= f.label "Name" %>
  <%= f.text_field :name, :value => @name %>
  <%= f.label "Start Date" %>
  <%= f.date_field :start_date %>
  <%= f.label "End Date" %>
  <%= f.date_field :end_date %>
  <%= f.submit %>
<% end %>
{% endhighlight %}

Now when the user clicks Create Trip the form is already populated with the campsite name. This is a simple example with a small amount of data that demonstrates how to make a better user experience by transferring data between views and populating forms with that data.

![filled form](/assets/filled_form.png)
