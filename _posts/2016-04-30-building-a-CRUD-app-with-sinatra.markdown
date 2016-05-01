---
layout: post
title: "Building A CRUD App with Sinatra"
date:   2016-04-30
categories: tutorial
---

This tutorial will go over building a fully functional CRUD (Create, Read, Update, Delete) application with Sinatra.  Following POODR tradition our application will be a bicycle related app.  The application will allow a user to create a bike project, i.e. Mountain Bike, set a budget, and add a list of parts with prices that will be needed to complete the project.  With this project we will be able to demonstrate how to implement each CRUD action within the Sinatra framework.  The complete project repo is available here: <a href="https://github.com/Smulligan85/Sinatra_Bike_Shop">Sinatra Bike Project</a>.

To begin we will set up our migrations and associations. Our app will have users, projects, and parts tables. Users will have many projects and have many parts through projects. Projects will have many parts and belong to a user. And finally Parts will belong to a project.  If you are familiar will Rails generators, Sinatra has a similar way of generating migrations with the help of `rake`. You can view all available rake commands with `rake -T`.  To create the migrations we can use the `rake db:create_migration NAME=create_table` where table is the table name.  Once we have the migrations we can run `rake db:migrate`.  Next we can create our three models with the appropriate ActiveRecord associations.

`user.rb`

{% highlight ruby linenos %}
class User < ActiveRecord::Base
  validates_presence_of :username, :email
  has_secure_password
  has_many :projects
  has_many :parts, through: :projects
end
{% endhighlight %}

`project.rb`

{% highlight ruby linenos %}
class Project < ActiveRecord::Base
  validates_presence_of :title, :budget
  belongs_to :user
  has_many :parts
end
{% endhighlight %}

`part.rb`

{% highlight ruby linenos %}
class Part < ActiveRecord::Base
  validates_presence_of :name, :price
  belongs_to :project
end
{% endhighlight %}

Once we have our models and associations set up we can set up our four controllers: ApplicationController, UsersController, ProjectsController and PartsController. These files will handle all of the routes and HTTP actions a user will need to maintain a specific project. They will also handle validations and redirect with appropriate error messages.

`application_controller.rb`

{% highlight ruby linenos %}
require './config/environment'

class ApplicationController < Sinatra::Base

  configure do
    set :public_folder, 'public'
    set :views, 'app/views'
    enable :sessions
    set :session_secret, "bike_security"
  end

  get '/' do
    erb :welcome
  end

  helpers do
    def logged_in?
      !!session[:user_id]
    end

    def current_user
      User.find(session[:user_id])
    end
  end

end
{% endhighlight %}

`users_controller.rb`

{% highlight ruby linenos %}
class UsersController < ApplicationController
  get '/signup' do
    if !logged_in?
      erb :'users/signup'
    else
      redirect to '/projects'
    end
  end

  post '/signup' do
    if params.values.any? {|value| value == ""}
      erb :'users/signup', locals: {message: "You can't do that!"}
    else
      @user = User.new(username: params[:username], email: params[:email], password: params[:password])
      @user.save
      session[:user_id] = @user.id
      redirect to '/projects'
    end
  end

  get '/login' do
    if !logged_in?
      erb :'users/login'
    else
      redirect to '/projects'
    end
  end

  post '/login' do
    user = User.find_by(:username => params[:username])
    if user && user.authenticate(params[:password])
      session[:user_id] = user.id
      redirect to '/projects'
    else
      erb :'users/login', locals: {message: "You credentials are incorrect!"}
    end
  end

  get '/logout' do
    if session[:user_id] != nil
      session.destroy
      redirect to '/'
    else
      redirect to '/projects'
    end
  end

end
{% endhighlight %}

`projects_controller.rb`

{% highlight ruby linenos %}
class ProjectsController < ApplicationController
  get '/projects' do
    if logged_in?
      @projects = Project.all
      erb :'projects/index'
    else
      erb :'users/login', locals: {message: "You don't have access, please login"} 
    end
  end

  get '/projects/new' do
    if logged_in?
      erb :'projects/new'
    else
      erb :'users/login', locals: {message: "You don't have access, please login"}
    end
  end

  post '/projects' do
    if params.values.any? {|value| value == ""}
      erb :'projects/new', locals: {message: "Your missing information!"}
    else
      user = User.find(session[:user_id])
      @project = Project.create(title: params[:title], budget: params[:budget], user_id: user.id)
      redirect to "/projects/#{@project.id}"
    end
  end

  get '/projects/:id' do 
    if logged_in?
      @project = Project.find(params[:id])
      erb :'projects/show'
    else 
      erb :'users/login', locals: {message: "You don't have access, please login"}
    end
  end

  get '/projects/:id/edit' do
    if logged_in?
      @project = Project.find(params[:id])
      if @project.user_id == session[:user_id]
       erb :'projects/edit'
      else
      erb :'projects', locals: {message: "You don't have access to edit this project"}
      end
    else
      erb :'users/login', locals: {message: "You don't have access, please login"}
    end
  end

  patch '/projects/:id' do 
    if params.values.any? {|value| value == ""}
      redirect to "/projects/#{params[:id]}/edit"
    else
      @project = Project.find(params[:id])
      @project.title = params[:title]
      @project.budget = params[:budget]
      @project.save
      redirect to "/projects/#{@project.id}"
    end
  end

  delete '/projects/:id/delete' do 
    @project = Project.find(params[:id])
    if session[:user_id]
      @project = Project.find(params[:id])
      if @project.user_id == session[:user_id]
        @project.delete
        redirect to '/projects'
      else
        redirect to '/projects'
      end
    else
      redirect to '/login'
    end
  end

end
{% endhighlight %}

`parts_controller.rb`

{% highlight ruby linenos %}
class PartsController < ApplicationController
  get '/projects/:id/parts/new' do
    if logged_in?
      @project = Project.find(params[:id])
      erb :'parts/new'
    else
      erb :'users/login', locals: {message: "You don't have access, please login"}
    end
  end

  post '/projects/:id' do
    if params.values.any? {|value| value == ""}
      @project = Project.find(params[:id])
      erb :'parts/new', locals: {message: "You are missing information!"}
    else
      @project = Project.find(params[:id])
      @part = Part.new(name: params[:name], price: params[:price])
      @part.save
      @project.parts << @part
      redirect to "/projects/#{@project.id}"
    end
  end

  delete '/projects/:id/parts/:part_id/delete' do 
    @project = Project.find(params[:id])
    @part = Part.find(params[:part_id])
    if logged_in?
      @project = Project.find(params[:id])
      if @project.user_id == session[:user_id]
        @part = Part.find(params[:part_id])
        @part.delete
        redirect to "/projects/#{@project.id}"
      else
        redirect to "/projects/#{@project.id}"
      end
    else
      erb :'users/login', locals: {message: "You don't have access, please login"}
    end
  end

end
{% endhighlight %}

With all of our logic and controller actions in place all we need to complete the application is create view pages for each get request action.
