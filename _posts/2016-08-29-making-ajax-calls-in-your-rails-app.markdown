---
layout: post
title: "Use AJAX to Retrieve Data in Your Rails Application"
date:   2016-08-28
categories: tutorial
---

This blog post will cover making AJAX calls to retrieve JSON data from your Rails application's controller actions.  We will be retrieving camping trip data from our example application.  An instance of the Trip class will have the following attributes: name, description, start date and end date.  To begin we will need to first have a way to serialize our trip data. Luckily Rails makes this very easy with the `active_model_serializers` gem.  Lets add this gem to our Gemfile and run `bundle install`.  Now we can utilize the gems generators to create our Trip Serializer.  In your terminal run `rails g seriailzer trip`.  This command will generate a serializers folder if it doesn't exist already and a file called `trip_serializer.rb`.  Next we need to update the attributes to add the attributes we want to serialize.

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

With the respond_to block we can determine whether the `@trips` data is rendered through the index view template or through JSON format depending on whether a html or json page is requested.  Next we will add a button that once clicked will display our trip data in a striped table. In the `views/users/show.html.erb` add:

{% highlight erb linenos %}
<div class="col-md-6">
  <h2>See Upcoming Trips</h2>
  <button data-user="<%= current_user.id %>" class="upcoming-trips btn btn-primary">See Upcoming Trips</button>
    <table class="table table-striped"></table>
</div>
{% endhighlight %}

Finally we can create a `trip.js` file to build our Trip object, add a helpful function to filter only trips occurring in the next 30 days and display those trips in the table.

{% highlight js linenos %}
function Trip(name, description, user_id, id, start_date, end_date) {
  this.name = name;
  this.description = description;
  this.user_id = user_id;
  this.id = id;
  this.start_date = start_date;
  this.end_date = end_date;
}

Trip.prototype.upcoming = function() {
  var todayDate = new Date();
  var monthDate = new Date();
  var oneMonthFromToday = new Date(monthDate.setMonth(todayDate.getMonth() + 1));
  var startDate = new Date(this.start_date);

  if (startDate >= todayDate && startDate <= oneMonthFromToday) {
    return true;
  } else {
    return false;
  }
};

// Function to retrieve index of Trips filtered with Trip.prototype.upcoming function
  $(".upcoming-trips").on("click", function() {
    $(".table-striped").html(
      "<tr>" +
      "<th>Trip Name</th>" +
      "<th>Start Date</th>" +
      "<th> End Date</th>" +
      "</tr>");
    var userId = $(this).data("user");
    $.get("/users/" + userId + "/trips.json", function(response) {
      $.each(response, function() {
        $.each(this, function(key, trip) {
          var tripData = new Trip(
            trip.name,
            trip.description,
            trip.user_id,
            trip.id,
            trip.start_date,
            trip.end_date
          );
          if (tripData.upcoming() === true) {
            $(".table").append("<tr>" +
                              "<td>" + tripData.name + "</td>" +
                              "<td>" + tripData.start_date + "</td>" +
                              "<td>" + tripData.end_date + "</td>" +
                              "</tr>"
            );
          }
        });
      });

    });
  });
{% endhighlight %}

Fantastic now when a user clicks the button "See Upcoming Trips" a table with all trips occurring in the next 30 days will appear and all without a page refresh!  I hope you found this post helpful.  If you have questions or thoughts, reach out on Twitter @seanmulligan85.
