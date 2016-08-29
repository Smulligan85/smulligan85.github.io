---
layout: post
title: "Use AJAX to Retrieve Data in Your Rails Application"
date:   2016-08-28
categories: tutorial
---

This blog post will cover making AJAX calls to retrieve JSON data from your Rails applications controller actions.  The data we will be retrieving will be camping trip data from our example application.  A trip instance will have the following attributes: name, description, start date and end date.  To begin we will need to first have a way to serialize our trip data. Luckily Rails makes this very easy with the active_model_serializers gem.  Lets add this gem to our Gemfile and run `bundle install`.  Now we can utilize the gems generators to create our Trip Serializer.  In your terminal run `rails g seriailzer trip`.  This command will generate a serializers folder if it doesn't exist already and a file called `trip_serializer.rb`.  Next we need to update the attributes to add the attributes we want to serialize.

{% highlight ruby linenos %}
class TripSerializer < ActiveModel::Serializer
  attributes :id, :name, :description, :start_date, :end_date
end
{% endhighlight %}

Now that we have our trip serializer set up we can format our index action to retrieve the collection of trips as JSON.  In `trips_controller.rb` add the following index action.

{% highlight ruby linenos %}
class TripsController < ApplicationController

  def index
    @trips = current_user.trips
    respond_to do |format|
      format.html { render :index }
      format.json { render :json => @trips }
    end
  end

end
{% endhighlight %}

With the respond_to block we can determine whether the `@trips` data is rendered through the index view template or through JSON format depending on whether a html or json page is requested.  Next we will quickly add a button show the user can click it and display all trip data. In the `views/users/show.html.erb` add:

{% highlight erb linenos %}
  <h2>See Upcoming Trips</h2>
  <button data-user="<%= current_user.id %>" class="upcoming-trips btn btn-primary">See Upcoming Trips</button>
  <table class="table table-striped">
    <tr id="headers">
      <th id="trip-name-head"></th>
      <th id="start-date-head"></th>
      <th id="end-date-head"></th>
    </tr>
    <tr id="table-data"></tr>
  </table>
{% endhighlight %}
