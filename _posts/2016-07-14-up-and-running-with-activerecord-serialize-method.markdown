---
layout: post
title: "Getting Up and Running With ActiveRecord Serialize Method"
date:   2016-07-14
categories: tutorial
---

On a recent project I came across a problem that I hadn't given much thought before. In Rails association of one class to another is pretty straight forward. For example, if you had a Dog class and a Dog instance had one instance of the Treat class, you can simply make this association via `has_one :treat` in your Dog class. This works great if you have complete control over the Treat class and can assign a dog_id to the treat database table. But what if the Treat class is outside of your root application, perhaps an object created by a third party gem that you want associated with your Dog class. This is where serialization comes in very handy. If we continue with our Dog class example, below we see a pretty basic Rails class.

{% highlight erb linenos %}
class Dog < ActiveRecord::Base
  serialize :treat
end
{% endhighlight %}

Now we just need add the treats column to our dog table.
{% highlight erb linenos %}
$ rails generate AddTreatToDog treats:text
$ rake db:migrate
{% endhighlight %}

It's important to assign the treats column as a text data type to insure `#serialize` works properly.

{% highlight erb linenos %}
fido = Dog.new
fido.treat = OutsideGem::Treat.new
{% endhighlight %}

That's it! Now we have access to all the methods available to the Treat class.

{% highlight erb linenos %}
fido.treat.healthiest #=> "Primal Pet Foods"
{% endhighlight %}

