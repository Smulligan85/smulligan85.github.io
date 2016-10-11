---
layout: post
title: "How to Use Components and Services in AngularJS"
date:   2016-10-10
categories: tutorial
---

One of the biggest aha moments I've had when using Angular was when I discovered the joy of using Components and Services together. When I initially began learning Angular I was taking a very Rails approach to the application architecture with a heavy reliance on routing to navigate to different view templates. With the help up Components to change the DOM within a single route and Services to interact with our Rails API backend we can be less reliant on routes to navigate our application. Let's get started!

First we will build out our Rails API TripsController that will communicate with out service.  Our controller will look like this:

{% highlight ruby linenos %}
module Api
  module V1
    class TripsController < ApplicationController

      def index
        respond_with(Trip.all.order("id ASC"))
      end

      def show
        respond_with(Trip.find(params[:id]))
      end

      def create
        trip = Trip.create(trip_params)
        if trip.save
          render :json => trip
        end
      end

      def update
        trip = Trip.find(params[:id])
        if trip.update(trip_params)
          render :json => trip
        end
      end

      def destroy
        respond_with(Trip.destroy(params[:id]))
      end

      private

      def trip_params
        params.require(:trip).permit(:name, :depart_date, :return_date)
      end
    end
  end
end
{% endhighlight %}

Now that we have our API set up we can create our TripService to communicate with it.

{% highlight javascript linenos %}
(function() {
  'use strict';

function TripService($http) {

  this.getTrips = function() {
    return $http.get('api/v1/trips.json');
  };

  this.getTripById = function(id) {
    return $http.get('api/v1/trips/' +id+ '.json');
  };

  this.newTrip = function(tripData) {
    return $http.post('api/v1/trips.json', tripData);
  };

  this.editTrip = function(id, updatedTripData) {
    return $http.put('api/v1/trips/' +id+ '.json', updatedTripData);
  };

  this.deleteTrip = function(id) {
    return $http.delete('api/v1/trips/' +id+ '.json');
  };
}

angular
  .module('app')
  .service('TripService', TripService);
}());
{% endhighlight %}

Our TripService contains a group of functions that communicate with our Rails API methods to retrieve, post, update, and delete data with the help of Angular's `$http` service. Now that we are able to communicate with our Rails API on the backend and TripService on the frontend we can build out our component and the corresponding html files. Let's start with the component.

{% highlight javascript linenos %}
(function() {
  'use strict';

  var individualTrip = {
    transclude: true,
    templateUrl: 'individual-trip.html',
    controller: IndividualTripController,
    bindings: {
      trip: '=',
      parentController: '='
    }
  };

  function IndividualTripController(TripService) {
    var ctrl = this;

    ctrl.readMode = true;
    ctrl.editMode = false;
    ctrl.editableTrip = ctrl.trip;
    ctrl.startEditMode = startEditMode;
    ctrl.closeEditMode = closeEditMode;
    ctrl.updateTrip = updateTrip;
    ctrl.destroyTrip = destroyTrip;

    function startEditMode() {
      ctrl.readMode = false;
      ctrl.editMode = true;
    }

    function closeEditMode() {
      document.location.reload(true);
      ctrl.editMode = false;
    }

    function updateTrip() {
      return TripService.editTrip(ctrl.editableTrip.id, ctrl.editableTrip)
              .success(function() {
                document.location.reload(true);
              });

    }

    function destroyTrip() {
      return TripService.deleteTrip(ctrl.editableTrip.id)
              .success(function() {
                document.location.reload(true);
              });
    }
  }
  angular
    .module('app')
    .component('individualTrip', individualTrip);
}());
{% endhighlight %}

First we declare a variable `individualTrip` to contain our component object. `individualTrip` will include transclude equal to true because we want our individual trip content to be nested inside a `<individualTrip>` tag. Next we set our html template. We will build this template shortly. We also set the `IndividualTripController` which is defined below the component object. And finally we set a binding so we have access to the individual trip and we bind our parentController which will have access to our collection of trips so we can iterate over the collection in our index html page. Below the component object we define our controller. Here we can set the variable `ctrl` to `this` and set a few attributes that we will have access to in our templates. Finally we have a series of functions that will both change our mode attributes and communicate with our `TripService`. Let's quickly build out our `index.html` file that will repeat over our collection and display each unique individualTrip template.

{% highlight html linenos %}
<div class="container">
  <h2>Your Trips</h2>
  <h3>Trip Name</h3>
    <div ng-repeat="trip in vm.trips | orderBy:'name'" title="{{trip.name}}">
    <individual-trip parent-controller="vm" trip="trip"></individual-trip>
    </div>
</div>
{% endhighlight %}

Now we can build out our individualTrip template that will display our trip data and links to edit and delete trips.

{% highlight html linenos %}
<div ng-if="$ctrl.readMode">
    <div class="col-md-6">
      <h2>{%raw%}{{$ctrl.trip.name}}{%endraw%}</h2>
      <p>Departing: {%raw%}{{ $ctrl.trip.depart_date | date: 'MM/dd/yyyy' }}{%endraw%}</p>
      <p>Returning: {%raw%}{{ $ctrl.trip.return_date | date: 'MM/dd/yyyy' }}{%endraw%}</p>
      <button class="btn btn-primary" ng-click="$ctrl.startEditMode()">Edit Trip      </button>
      <button class="btn btn-danger" ng-click="$ctrl.destroyTrip()">Delete Trip</button><br>
    </div>
</div>

<div ng-if="$ctrl.editMode">
  <h2>Edit Your Trip</h2>
  <button class="btn btn-danger" ng-click="$ctrl.closeEditMode()">Close Edit Form</button>
  <form  name="form" novalidate ng-submit="$ctrl.updateTrip()">
    <div class="form-group">
    <label for="name">Name:</label>
    <input class="form-control" type="text" name="tripName" placeholder="{{$ctrl.trip.name}}" ng-model="$ctrl.trip.name" required ><br>
      <div ng-messages="form.tripName.$error">
        <div ng-message="required" ng-if="form.$submitted">Trip name is required.</div>
      </div>
    </div>

    <div class="form-group">
    <label for="depart">Depart Date:</label>
    <input class="form-control" type="date" name="depart" ng-model="$ctrl.trip.depart_date" required ><br>
      <div ng-messages="form.depart.$error">
        <div ng-message="required" ng-if="form.$submitted">Depart date is required.</div>
      </div>
    </div>

    <div class="form-group">
    <label for="return">Return Date:</label>
    <input class="form-control" type="date" name="return" ng-model="$ctrl.trip.return_date" required ><br>
      <div ng-messages="form.return.$error">
        <div ng-message="required" ng-if="form.$submitted">Return date is required.</div>
      </div>
    </div>

    <input class="btn btn-primary" type="submit" value="Update Trip">
  </form>
    </div>
</div>
{% endhighlight %}

With this template we will have full access to each individual trip and be able to edit or delete without created separate routes or controllers for those actions. I hope you found this helped to unravel how components and services can make Angular development enjoyable and productive. Feel free to reach out on Twitter with any questions.
