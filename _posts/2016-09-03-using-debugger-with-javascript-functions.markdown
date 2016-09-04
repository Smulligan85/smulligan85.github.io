---
layout: post
title: "Using Debugger to Examine Javascript Functions"
date:   2016-09-03
categories: tutorial
---

As a Rails developer one of my favorite gems for debugging code is `pry`.  As you may know the `pry` gem allows you to set a breakpoint by placing a `binding.pry` within the code. Once the breakpoint is run a REPL session will begin and you can play around with variable assignment and general logic to discover bugs.  As I've dived deeper into Javascript, I've found myself many times wishing I had the ability to pause running the program and play around with a function.  This tutorial will cover a built in tool that Javascript provides for breaking out of a running program, simply called `debugger`.

Using `debugger` is extremely simple and allows you to run a REPL session within your developer tools console.  To demonstrate, here we have a small hello world function that handles a click event with a little help from JQuery and alerts the user with the message "Hello World".  Add the following code to a file called `index.html`.

{% highlight html linenos %}
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
</head>
<button class="hello-world">Click Me!</button>
<script>
$('.hello-world').on('click', function() {
  var hello = "Hello World";
  alert(hello);
});
</script>
{% endhighlight %}

Now if we open up `index.html` in our browser we should be able to click the button and cause a alert to pop up with the text "Hello World".  However, lets imagine for this example that the alert did not trigger or that the text was not what we had expected.  This is where `debugger` comes in.  Update the code by adding the `debugger` breakpoint.

{% highlight html linenos %}
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
</head>
<button class="hello-world">Click Me!</button>
<script>
$('.hello-world').on('click', function() {
  var hello = "Hello World";
  debugger;
  alert(hello);
});
</script>
{% endhighlight %}

Similar to `pry` we can place the `debugger` anywhere within our Javascript code so we can pinpoint the problem more efficiently. Now before we trigger the debugger breakpoint it is important to note that we must have our developer tools open to trigger `debugger`.  Without developer tools open the program will run normally without hitting the `debugger` breakpoint.  

Now with the `debugger` in place when we click the button you will notice the program pause and access our developer tools console.  We also have access to all variables assigned prior to the breakpoint, i.e. the hello variable.  Not only can we access the variable, but we can also reassign it and continue the program.  Lets reassign the hello variable in the console, `hello = "Goodnight World";`  To continue the program click the play button in the center of the browser.  Now the alert will contain the text from the reassigned variable.

I hope this tutorial gave you a helpful overview of how to use `debugger` to inspect your Javascript code.  I recommend placing `debugger` breakpoints within more complex functions and playing around with the logic.  If you have questions or thoughts, reach out on Twitter @seanmulligan85.
