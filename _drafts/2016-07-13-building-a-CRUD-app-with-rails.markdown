---
layout: post
title: "Building A CRUD App with Rails"
date:   2016-07-13
categories: tutorial
---

Today we're going to go over building a simple Rails CRUD application, use the Active API to retrieve data with the faraday gem, and implement omniauth log in via Twitter. The name of our application will be call CampHunt and the main purpose is to allow users to search for parks, such as Yosemite and save a campsite trip, along with a date and a list of supplies. The basic user flow will go as follows:

1. User signs up or logs in with email/password or via Twitter log in credentials.
2. Upon log in User is redirected to their unique homepage that includes a search field to search for new trips and a list of trips coming up within the next 30 days.
3. When a User searches for a park, i.e. Yosemite, the User is redirected to a results page that lists out each campground with an accompanying button to create a new trip.
4. Clicking Create Trip redirects the User to a new form with the campground name and description prepopulated courtesy of the Active API.
5. The User can then edit the name, description and add the start and end dates.
6. Once the trip is saved a User can then optionally add a list of supplies needed for the trip, i.e. tent, sleeping bag.
