---
layout: post
title: "Common Security Issues in Rails"
date:   2016-11-04
categories: tutorial
---

When building a web application security should be a top concern to any development team. The Rails framework takes security vulnerabilities into account and provides helpful methods and conventions to secure your application. This post will cover two aspects of security within a Rails application: SQL Injection and Sessions.

The first issue we will cover will involve SQL Injection and how Rails prevents these forms of the attack. SQL Injection is a common technique that attempts to influence database queries by passing malicious strings as a parameter. The goal could be to simply bypass authorization or retrieve and manipulate data. Applying the following ActiveRecord query into your application opens your application up to this form of attack: `Post.where("title = '#{params[:title]}'")`. This allows an attacker to simply pass a malicious string of SQL code that is then interpolated by the where clause to retrieve and manipulate information that should not be accessible. Rails has many built in methods that automatically counter this form of attack, such as using `Post.find(params[:id])`. However, SQL fragments such as where clauses still require you to apply countermeasures manually. Instead of passing a string as we did with the above where clause. We must pass an array to sanitize any malicious strings, `Post.where("title = ?", post_title)`. The sanitized version of the variable then replaces the question mark in the SQL condition. 

Another area vulnerable to attacks involves how your application handles sessions. HTTP is a stateless protocol that treats each request independently. Thus, sessions allow an application to persist data between multiple HTTP requests. For example, if a user adds items to a shopping cart, the contents of the cart should persist throughout the lifecycle of shopping session. Rails creates a new session automatically if a new user accesses the application or loads a previous session if the user has already used the application. A session consists of a hash where the session id is set to a 32 byte random string value. If you are using a authentication systems such as Devise the session will automatically expire when a user signs in or signs out. However, if you are rolling out your own authentication it is important to implement a way to expire sessions. Sessions that never expire extend the time vulnerability of your application. A way to require expiration of the session id is to define a method that automatically clears the session after a certain amount of time of inactivity.

{% highlight ruby linenos %}
class Session < ApplicationRecord
  def self.sweep(time = 1.hour)
    if time.is_a?(String)
      time = time.split.inject { |count, unit| count.to_i.send(unit) }
    end

    delete_all "updated_at < '#{time.ago.to_s(:db)}' OR
      created_at < '#{2.days.ago.to_s(:db)}'">>"
  end
end
{% endhighlight %}

By calling `Session.sweep` the session will expire after 1 hour of inactivity from either the `updated_at` or `created_at` time.

I hope that you found this post helpful in explaining some common security vulnerabilities and how Rails helps to counter attacks. Feel free to reach out on Twitter with any questions.
