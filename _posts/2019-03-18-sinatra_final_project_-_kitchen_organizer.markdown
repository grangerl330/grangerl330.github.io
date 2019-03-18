---
layout: post
title:      "Sinatra Final Project Kitchen Organizer"
date:       2019-03-18 14:25:26 -0400
permalink:  sinatra_final_project_-_kitchen_organizer
---

Today I completed my final project for the sinatra section of the Flatiron Online Software Engineering Course. My project is called Kitchen Organizer. It is an application that allows the user to organize items in their kitchen into different cabinets that can be labeled by name and total capactiy. It is a simple content managment system that covers the main themes learned in the sinatra section, CRUD (Create, Read, Update, Delete) database management, and MVC (Model, View, Controller) application design. 


## Creating the Models

My first step in completeing this project was creating the models that my application would rely on. These models include the `User`, `Cabinet`, and `Item` classes. These classes each have relationships with each other that determine how they interact in the database. We can use `ActiveRecord`, a Ruby database management gem, to establish these relationships as follows:

```
class User < ActiveRecord::Base
  has_many :cabinets
  has_many :items
	has _secure_password
end
```

A `User` has many `Cabinets` and has many `Items`. `has_secure_password` is a feature from a gem called `bcrypt` that I will explain in more detail later. 

```
class Cabinet < ActiveRecord::Base
  belongs_to :user
  has_many :items
end
```

A `Cabinet` belongs to a `User` and has many `Items`

```
class Item < ActiveRecord::Base
  belongs_to :cabinet
  belongs_to :user
end
```

An `Item` belongs to a `Cabinet` and belongs to a `User`

These relationships effect how the data for each model is tracked when stored in the database and will allow us to call on certain attributes of each class in the future. For example, because an instance of user has many cabinets, we have the ability to call `user.cabinets`, where user is an instance of the `User` class, to see all of that user's cabinets that he or she has created. We could also call `cabinet.user` to see a cabinet's user, because a cabinet belongs to a user. These relationships provided by `ActiveRecord` allow for much simpler data manipulation when programming.

In addtion to the listed relationships, each class inherits from `ActiveRecord::Base`. This inheritance provides the methods `belongs_to` and `has_many` to each of the classes. 


## Creating the Database

After creating the models and establishing their connections with each other, it was time to create the actual database the data for each model would be stored in. To do this, we use a gem called Rake, combined with Active Record, which provides us methods from creating database tables.

First I needed to create a folder called `db` to house all information about the database. Next I created a folder called `migrate` that would contain an individual migration file for each of the tables created, one table for each of the models:

```
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest
    end 
  end
end
```

The `users` table has columns for `username` and `password_digest`. `username` is obviously self-explanatory. `password digest`  is part of the `bcrypt` gem. This gem provides encryption for passwords in a database. When a user signs up with a password, that password is encrypted and stored in the database as an encrypted value. This provides an extra layer of security. If an unauthorized person ever gets access to the applications database, they will not be able to see the users' passwords, only the passwords' encrypted values. 

```
class CreateCabinets < ActiveRecord::Migration
  def change
    create_table :cabinets do |t|
      t.string :name
      t.integer :capacity
      t.integer :user_id
    end
  end
end
```

The `cabinets` table has columns for `name`, `capactiy`, and `user_id`. `user_id` is added because a `cabinet` belongs_to a `user`. When we use an `ActiveRecord` method to find a `user`'s cabinets, it will look for all cabinets with a `user_id` equal to the id of the `user`

```
class CreateItems < ActiveRecord::Migration
  def change
    create_table :items do |t|
      t.string :name
      t.integer :cabinet_id
      t.integer :user_id
    end
  end
end
```

The `items` table has columns name, `cabinet_id`, and `user_id`. These id columns are for the functionality of the `ActiveRecord` methods, just like the `user_id` in the `cabinets` table

After each migration file is created, we can run the command `rake db:migrate` and a table for each class with each of the listed properties will be created and stored in the database. 


## Building The Controllers

The next step was creating the `contoller` classes that are responsible for directing the program to a specifc view for each URL that the user can visit. I separated these responisbilites out into 4 different controller classes, `application_controller`, `users_controller`, `cabinets_controller`, and `items_controller`. 

Each controller class is responisble for the URL that relate to that controller's name. For example, the URL `/items/new` which is the page that allows a user to create a new item, is in the `items_controller` class. Separating specifc URL's into specific controller classes allows for much easier program management because we will always know what file to look in if there is some sort of error on an individual page. 

Below I will display the code for each fo the controller classes and highlight specific sections that display common themes throughout the application:

###  Application Controller

```
require './config/environment'

class ApplicationController < Sinatra::Base

  configure do
    set :public_folder, 'public'
    set :views, 'app/views'
    enable :sessions
    set :session_secret, "kitchen_organizer_secret"
  end

  get '/' do
    if logged_in?
      redirect '/cabinets'
    else
      redirect '/login'
    end
  end

  get '/welcome' do
    erb :'/welcome'
  end

  helpers do
    def redirect_if_not_logged_in
      if !logged_in?
        redirect "/login?error=You have to be logged in to do that"
      end
    end

    def logged_in?
      !!session[:user_id]
    end

    def current_user
      User.find(session[:user_id])
    end
  end

end
```

This controller is used for the main "set up" of the application and renderring the login and signup pages. Here you can see that sessions are enabled, which provides a way for a web application to set a cookie that persists an identifier across multiple HTTP requests, and then relate these requests to each other. 

Also, a set of helper methods are defined, which can be accessed in other parts of the application at any time to make programming much easier. Here a method `logged_in?` which allows us to check if the user is logged in, `redirect_if_not_logged_in` which will redirect the user to the login page if they are not logged in, and `current_user` which returns the `User` instance that is the current user are all defined. 

The `get  '/'` route provides the execution that will occur when the user visits the '/' route of the application. Here we can see the `logged_in?` method in action. If the user is logged in, they will be redirected to the `/cabinets` route, if they are not, they will be redireced to the `/login` route. 

*This is a common theme with all of the controllers, as it is almost always necessary to ensure that a user is logged in before allowing them access to a specific route

The `get '/welcome'` route provides the execution that occurs when a user naviagtes to the '/welcome' URL. This renders a view, which is an erb file that contains the HTML that will be displayed for this route. 

### Users Controller

```
class UsersController < ApplicationController

  get '/signup' do
    if logged_in?
      redirect '/cabinets'
    else
      erb :'/users/new'
    end
  end

  post '/signup' do
    if params.value?("")
      redirect '/signup'
    else
      @user = User.create(params)
      session[:user_id] = @user.id
      redirect '/welcome'
    end
  end

  get '/login' do
    if logged_in?
      redirect '/cabinets'
    else
      erb :'/users/login'
    end
  end

  post '/login' do
    @user = User.find_by(:username => params[:username])
    if @user && @user.authenticate(params[:password])
      session[:user_id] = @user.id
      redirect '/cabinets'
    else
      redirect '/login?error=Invalid username or password'
    end
  end

  get '/logout' do
    if logged_in?
      session.clear
      redirect '/'
    else
      redirect '/'
    end
  end

end
```

This controller is responsible for all of the routes that have to do with the `User` class model. It contains all of the routes for login, signup, and logout. For each of the `get` routes, when a user navigates to that URL, a proccess of execution is written that ends in the rendering of an erb file. These erb files are the views, and contain the HTML that is actually displayed when the user visits those URL's. 

The `post` routes are for when a user enters information on one of the `get` routes, such as a username and password on the `/login` page. These `post` routes take that information entered by the user and perform and action with it, such as creating a new instance of the `User` class and setting the `session[:user_id]` equal to the newly created user's id in the `post /signup` route. 

This process occurs multiple times throughout the application: A user navigates to a page via a `get` method, enters information into a form which is displayed via the HTML in the corresponding views erb file, and the information is then used to peform some sort of action via a `post` method. This pattern provides nearly all of the functionality of this application

### Cabinets Controller

```
class CabinetsController < ApplicationController

  get '/cabinets' do
    redirect_if_not_logged_in
    @user = current_user
    erb :'/cabinets/index'
  end

  get '/cabinets/items' do
    redirect_if_not_logged_in
    @user = current_user
    erb :'/cabinets/index_with_items'
  end

  get '/cabinets/new' do
    redirect_if_not_logged_in
    erb :'cabinets/new'
  end

  post '/cabinets' do
    if params.value?("")
      redirect '/cabinets/new'
    elsif params["capacity"].to_i == 0
      redirect '/cabinets/new'
    else
      @cabinet = Cabinet.create(params)
      @cabinet.user_id = session[:user_id]
      @cabinet.save
      redirect "/cabinets"
    end
  end

  get '/cabinets/:id' do
    redirect_if_not_logged_in
    @cabinet = Cabinet.find_by_id(params[:id])
    if @cabinet.user_id == session[:user_id]
      erb :'/cabinets/show'
    else
      redirect '/cabinets'
    end
  end

  get '/cabinets/:id/edit' do
    redirect_if_not_logged_in
    @cabinet = Cabinet.find_by_id(params[:id])
    if @cabinet.user_id == session[:user_id]
      erb :'/cabinets/edit'
    else
      redirect '/cabinets?error=You do not have access to this cabinet'
    end
  end

  post '/cabinets/:id' do
    @cabinet = Cabinet.find_by_id(params[:id])

    if params["name"] == "" && params["capacity"] == ""
      @cabinet
    elsif params["name"] == ""
      @cabinet.update(capacity: params["capacity"])
    elsif params["capacity"] == ""
      @cabinet.update(name: params["name"])
    else
      @cabinet.update(params)
    end
    redirect "/cabinets/#{@cabinet.id}"
  end

  delete '/cabinets/:id/delete' do
    @cabinet = Cabinet.find_by_id(params[:id])

    @cabinet.items.each do |item|
      item.destroy
    end
    @cabinet.destroy

    redirect '/cabinets'
  end

end
```

This controller provides the functionality for all of the routes that relate to the `Cabinet` class.

Here we can see that the `redirect_if_not_logged_in` method that was defined in the `Application Controller` is used in every `get` method. This is making sure that the user is currently logged in before providing he or she acces to the specified route within the application. 

Each `/get` route is performing the same basic function, setting an instance variable `@user` equal to the current user and then renderring an erb file.  We set the `@user` variable so that we can pass that information into the erb file that is being rendered and display it in the HTML. The only way information can be passed from the controller into an erb file is through an instance variable. This process is also used to set the `@cabinet` variable when we need to display information about a cabinet in an erb file. 

The post methods in this controller both follow the same pattern. First they check the information that ws entered into the form by the user. This information is avaiable through the `params` hash. If the information entered passes the conditions, which are mainly checking if the user entered valid inputs into the proper fields on whatever page the post is coming from, an instance of the `Cabinet` class will either be created or updated using the information that was entered by the user. Then the user will be redirected to another route. 

The delete method do exactly what you think, delete the specified cabinet. This method also iterates through the cabinet's item and deletes each one as well. Then it redirects the user to the `/cabinets` route

### Items Controller 

```
class ItemsController < ApplicationController

  get '/items/new' do
    @cabinet = Cabinet.find_by_id(params["cabinet id"])
    redirect_if_not_logged_in
    @user = current_user
    erb :'items/new'
  end

  post '/items' do
    @user = current_user
    @cabinet = Cabinet.find_by(name: params["cabinet name"], user_id: @user.id)
    if params["name"] == ""
      redirect '/items/new?error=Item must have a name'
    else
      @item = Item.create(name: params[:name], cabinet_id: @cabinet.id, user_id: @user.id)
    end

    redirect "/cabinets/#{@cabinet.id}"
  end

  get '/items/:id/edit' do
    redirect_if_not_logged_in
    @item = Item.find_by_id(params[:id])
    @user = current_user

    if @item.user_id == session[:user_id]
      erb :'/items/edit'
    else
      redirect '/items?error=You do not have access to this item'
    end
  end

  post '/items/:id' do
    @item = Item.find_by_id(params[:id])
    @cabinet = Cabinet.find_by_name(params["cabinet name"])

    if params["name"] == ""
      redirect "/items/#{@item.id}/edit?error=Name must be filled in"
    else
      @item.update(name: params["name"], cabinet_id: @cabinet.id)
    end

    redirect "/cabinets/#{@cabinet.id}"
  end

  delete '/items/:id/delete' do
    @item = Item.find_by_id(params[:id])
    @cabinet = @item.cabinet

    @item.destroy

    redirect "/cabinets/#{@cabinet.id}"
  end

end

```

The items controller is provides the functionality for all of the routes that relate to the `Item` class. 

This controller follows the same logic as the `Cabinets Controller`. The `get`, `post`, and `delete` routes all provide very similar functionality, they are just manipulating data about instances of `Item` instead of `Cabinet`.


## Writing The Views

The views files are all of the erb files that are referrenced in the `get` routes in the controllers. Each `get` route renders its own HTML file which provides that code that is actually display in the browser when a user navigates to that URL. erb files are used because they are a type of file that allows the programmer to write HTML and insert blocks of Ruby code into that HTML to provide additional functionality. 

To avoid redundancy,  I will go over only two of the cabinets views that display the common patterns seen in all of the views files: 

### cabinets/new.erb

```
<h2>Create A New Cabinet</h2>

<form action="/cabinets" method="POST">
  <p style="margin-bottom: 0">Name: <input type="text" name="name" required></p>
  <p style="color:#b8b8b8; font-size:12px;">Ex: Spices, Canned Goods</p>
  <p style="margin-bottom: 0">Capacity: <input type="text" name="capacity" required></p>
  <p style="color:#b8b8b8; font-size:12px;">Number of items that can fit in the cabinet</p>
  <p style="margin-bottom: 0"><input type="submit" value="Create Cabinet"></p>
</form>

```

This file is the view that is rendered when a user navigates to the `/cabinets/new` URL. It contains the above HTML, which displays an `<h2>` header and a `<form>` which allows the user to enter information that the `post` route will use to perform an action. This form contains two `<input>`s that the user can type information into and an `<input>` that is a submit button. So when the user is directed to this page, her or she can enter a cabinet name and capacity, and press the submit buttom. When the submit button is pressed the `post /cabinets` route, which is specified in the first line of the form,  will run with the information the user entered stored in the `params` hash.  The `params` keys will be the values next to `name=` in each input, and the values corresponding to those keys will be whatever the user typed into the inputs before pressing submit. 

### cabinets/index.erb

```
<h2>Your Kitchen
  <span style="font-size:15px; color:#b8b8b8; font-weight:normal">
    <i><%=@user.cabinets.count%>
      <% if @user.cabinets.count == 1 %>
        cabinet,
      <% else %>
        cabinets,
      <% end %>
      <%=@user.items.count%>
      <% if @user.items.count == 1 %>
        item
      <% else %>
        items
      <% end %>
    </i>
  </span>
</h2>

<% @user.cabinets.each do |cabinet| %>
  <a href="/cabinets/<%=cabinet.id%>"><h3><%=cabinet.name %></h3></a>
  <p style="color:#b8b8b8">Capacity:
    <% if cabinet.items.count > 0 %>
      <%= cabinet.items.count %>/<%= cabinet.capacity %></p>
    <% else %>
      Empty
    <% end %>
  <hr>
<% end  %>

<p style="margin-top: 20px"><a href="/cabinets/new"><input type="button" value="Add Cabinet"></a></p>
```

This view is rendered when the user navigates to the `/cabinets` route. It displays all of the cabinets associated with the current user. 

This erb file makes use of the `@user` instance variable that was set in the `Cabinets Controller` `get /cabinets` route. As you can see, whenever the `@user` instance variable is referenced, it is placed within either a `<%= %>` tag or a `<% %>` tag. This is how we specify to the interpreter what information is Ruby code and what is HTML. Anything inside one of these tags is designated as Ruby and will function just like normal Ruby code. The `<%= %>` tag is sort of like string interpolation `#{}` in Ruby. It essentially displays the value of whatever Ruby code is inside it as if you were just typing that value directly in to the HTML. In this view we can see these tags are used to iterate through all of the user's cabinets and display each cabinet name and capactiy. 

## Conclusion 

I had a great time working on this project. Although it was challenging at times, I really enjoyed putting all of the information I had learned in the Sinatra section of the course to use to build an actual functioning application. As I'm sure many others do, I learn best by actually working on projects that require me to use the new information I am being taught. After completing this project, I feel that I have a much better understanding of the the basic structure of an MVC application, and I am ready to move on to the next step of creating more complicated applications with more advanced functionality









