---
layout: post
title: "Building A CRUD App with Rails, Faraday and Devise"
date:   2016-07-13
categories: tutorial
---

Today we're going to go over building a simple Rails CRUD application. Specifically, the first part of this tutorial will go over how to set up our initial landing page, install and configure authentication with Devise, and use the Active API to retrieve data with the faraday gem. The name of our application will be called CampHunt and the main purpose is to allow users to search for state and national parks and save a camping trip, along with a date, description, and a list of supplies. The basic user flow will go as follows:

1. User signs up or logs in with email and password.
2. Upon log in User is redirected to their homepage that includes a search field to search for new trips and a list of trips coming up within the next 30 days.
3. When a User searches for a park, i.e. Yosemite, the User is redirected to a results page that lists out each campground with an accompanying button to create a new trip.
4. Clicking Create Trip redirects the User to a new form with the campground name and description prepopulated courtesy of the Active API.
5. The User can then edit the name, description and add the start and end dates.
6. Once the trip is saved a User can then optionally add a list of supplies needed for the trip, i.e. tent, sleeping bag.

## Setting Up Our Application

Ok lets get started by initializing our project with: `rails new camphunt`. Before we run `bundle install` lets add a few gems we will need later on. In your Gemfile add the following lines.

{% highlight ruby linenos %}
gem 'bootstrap-sass'
gem 'devise'
gem 'faraday', '~> 0.9.2'
{% endhighlight %}

To begin we will first set up our landing page and add some basic CSS. We can use the `bootstrap-sass` gem we installed to add styles easily into our application. Lets quickly configure bootstrap to work with our application. First, open up our `application.js` file and add `//= require bootstrap-sprockets` just below the line requiring turbolinks. Then rename our application.css file to application.scss. This will allow us to use sass in our css file. Finally, lets open up our `application.scss` file and add the following code.

{% highlight css linenos %}
@import "bootstrap-sprockets";
@import "bootstrap";

body {
  background: #F8F8FF;
  }

nav.navbar {
  background: #F8F8FF;
  border: none;
}

.nav.navbar-nav.nav-text li a {
  font-size: 25px;
  font-weight: bold;
  color: #75D701;
}
{% endhighlight %}

We will also create a `landing.scss` file to add some CSS specific to our landing page. Open up `landing.scss` and add the following.
{% highlight css linenos %}
.landing-text {
  text-align: center;
  color: #56A902;
  h1 {
    font-size: 100px;
    font-weight: bold;
  }
  h2 {
    font-size: 50px;
    font-weight: bold;

  }
}
{% endhighlight %}

Now that we have our CSS set up. Lets move on to the controller and view page for the landing section of our application. Create a `landing_controller.rb` within your controllers directory and within our views directory create a new directory called landing. Within views/landing/ create a file called `index.html.erb`. Now lets open up our `landing_controller.rb` file and add the following code.

{% highlight ruby linenos %}
class LandingController < ApplicationController
  def index
  end
end
{% endhighlight %}

Next lets set up our index route as our `root_path`. Open up `routes.rb` and add `root 'landing#index'`. Now that we have our index action created we can open up our `views/layout/index.html.erb` and add some HTML to view on our page. However, before we add that lets create a simple navigation and some flash notifications that will be shared across all view pages. Open up `view/layout/application.html.erb` and add the following.
{% highlight erb linenos %}
<!DOCTYPE html>
<html>
<head>
  <title>CampHunt</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="collapse navbar-collapse">
      <ul class="nav navbar-nav nav-text">
        <li><%= link_to "Camp Hunt", root_path %></li>
      </ul>
    </div>
  </div>
</nav>
<% if flash[:notice] %>
  <div class="alert alert-success">
    <button type="button" class="close" data-dismiss="alert">&times;</button>
    <%= flash[:notice] %>
  </div>
<% elsif flash[:error] %>
  <div class="alert alert-danger">
    <button type="button" class="close" data-dismiss="alert">&times;</button>
    <%= flash[:error] %>
  </div>
<% elsif flash[:alert] %>
  <div class="alert alert-warning">
    <button type="button" class="close" data-dismiss="alert">&times;</button>
    <%= flash[:alert] %>
  </div>
<% end %>
<%= yield %>

</body>
</html>
{% endhighlight %}

## Install and configure Devise Authentication

Now that we have a landing page we can install and configure our authentication system with Devise. To get started run `rails generate devise:install`. Next we'll handle some quick configuration. Open up `config/environments/development.rb` and add `config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }`. Finally, we can generate our User class with devise. Run `rails generate devise User` followed by `rake db:migrate` to migrate the database. Finally, we can also generate the basic log in views with Devise too. Run `rails generate devise:views`. Now we have Devise set up!

Next we will do some house cleaning to add routes and links to our navigation bar. First, open up `routes.rb` and add the following lines.
{% highlight ruby linenos %}
  devise_for :users

  authenticated :user do
    root 'users#show', :as => :authenticated_root
  end
  root 'landing#index'
{% endhighlight %}

For our authenticated route lets create a `users_controller.rb` file and add the following.
{% highlight ruby linenos %}
class UsersController < ApplicationController
  before_action :authenticate_user!

  def show
    @user = current_user
  end

end
{% endhighlight %}

And we can create a `views/users/show.html.erb` template to display the user email. Later we will be adding the camping search field here as well.

{% highlight ruby linenos %}
<h1>Welcome <% @user.email %></h1>
{% endhighlight %}

And we will also update our `application.html.erb` file to insert links for our new routes. Update the nav block with the following.

{% highlight erb linenos %}
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="collapse navbar-collapse">
      <ul class="nav navbar-nav nav-text">
        <li><%= link_to "Camp Hunt", root_path %></li>
        <% if current_user %>
          <li><%= link_to "Log Out", destroy_user_session_path, method: :delete %></li>
        <% else %>
          <li><%= link_to "Sign In", new_user_session_path %></li>
          <li><%= link_to "Sign Up", new_user_registration_path %></li>
        <% end %>
      </ul>
    </div>
  </div>
</nav>
{% endhighlight %}

Next we will get our Trip and Supply models set up. We will also create a join table called trip_supplies table for handling our nested form. Lets begin by generating our migration in terminal. Run the following lines:
 `rails generate migration CreateTrips name:string start_date:date end_date:date description:text contract_code:string park_id:integer references:users`
`rails generate migration CreateSupplies name:string`
`rails generate migration CreateTripSupply trip_id:integer supply_id:integer`
`rake db:migrate`

Now we can create our models with the appropriate associations and validations.

{% highlight ruby linenos %}
class Supply < ActiveRecord::Base
  belongs_to :trip
  has_many :trip_supplies
  has_many :trips, :through => :trip_supplies
  validates :name, :presence => true
end
{% endhighlight %}


{% highlight ruby linenos %}
class Trip < ActiveRecord::Base
  belongs_to :user
  has_many :trip_supplies
  has_many :supplies, :through => :trip_supplies

  validates :name, :presence => true
  validates :start_date, :presence => true
  validates :end_date, :presence => true
end

{% endhighlight %}

{% highlight ruby linenos %}
class TripSupply < ActiveRecord::Base
  belongs_to :trip
  belongs_to :supply
end
{% endhighlight %}

## Building Our Campsite Search Feature

Before we can create a new camping trip we need to build out a search feature and connect to the Active API to retrieve camping data. Visit the [Active API developer page](http://developer.active.com/) and create an account. Once you are logged in you can create an application and retrieve your API_KEY. Once you have that set up lets open up yoru `.gitignore` file and add the following line `config/secrets.yml`. Now we can open up out secrets file and add our API keys under development so we have access to them in our application.

`active_api_key: <Replace with your API key>`

Next lets create our a results action that will parse the form params and connect with our Active API. We'll be adding this to our `trips_controller.rb` that we'll create now.

{% highlight ruby linenos %}
class TripsController < ApplicationController

  def results
    response = Faraday.get do |req|
      req.url "http://api.amp.active.com/camping/campgrounds/?"
      req.params['api_key'] = Rails.application.secrets.active_api_key
      req.params['pname'] = params["pname"]
    end
    @json = JSON.parse(Hash.from_xml(response.body).to_json)
    if @json["resultset"]["count"] == "0"
      flash[:error] = "Park not found"
      redirect_to root_path
    end
  end

end
{% endhighlight %}

In order to pass the results method a params hash we'll have to build out our form. Open up our user show view and add the following form.

{% highlight ruby linenos %}
<div class="col-md-6">
  <h2>Find a new adventure</h2>
  <%= form_tag("trips/results", method: "post") do %>
    <%= label_tag("Park Name") %>
    <%= text_field_tag(:pname) %>
    <%= submit_tag("Search") %>
  <% end %>
</div>
{% endhighlight %}

Finally we can create a view in `views/trips/results.html.erb` to display the json data.

{% highlight ruby linenos %}
<h1>Here are your results for <%= @json["resultset"]["pname"].titlecase %></h1>

<table class="table table-striped">
  <% if @json["resultset"]["result"].is_a?(Array) %>
    <% @json["resultset"]["result"].each do |park| %>
      <tr>
        <td>
          <%= park["facilityName"].titlecase %></br>
        </td>
      </tr>
    <% end %>
  <% else %>
      <tr>
        <td>
          <%= @json["resultset"]["result"]["facilityName"].titlecase %><br>
        </td>
      </tr>
  <% end %>
</table>
{% endhighlight %}

Awesome! We are now able to search for a national or state park and return a table of campsites within that park via the Active API! To recap our Rails tutorial part one covered setting up our basic landing page, installing and configuring authentication with Devise, and building our campsite search feature. Part two will go over adding data such as name, dates and supplies to our trips.
