# RESTful Routes

## Overview

In this lesson we'll explain the benefits of RESTful routes and how they provide a design pattern that allows for easy data manipulation.

## Objectives
+ Explain the concept of RESTful routes
+ Implement RESTful routes in a Sinatra application

## What Is A RESTful Route?

The internet would be a really confusing place without a convention for how to handle URLS - to delete an Instagram photo might be www.instagram.com/delete-this-photo, but Twitter might be www.twitter.com/remove-this-tweet. Without a specific convention to follow, it would be hard to create new content, edit content, and delete it. RESTful routes provides a design pattern that allows for easy data manipulation. It's nicer for users and nicer for developers to have everything consistent.

A RESTful route is a route that provides mapping between HTTP verbs (get, post, put, delete, patch) to controller CRUD actions (create, read, update, delete). Instead of relying solely on the URL to indicate what site to visit, a RESTful route also depends on the HTTP verb __and__ the URL.

What this means is that when your application receives an HTTP request, it introspects on that request and identifies the HTTP method and URL, connects that with a corresponding controller action that has that method and URL, executes the code in that action, and determines which response gets sent back to the client. We don't need to worry about how the mechanics of the pattern matching occurs, just that it does happen.

It's important to note that much of the CRUD actions are different actions that occur on the same resource. Let's take the example of an article with the ID 4. If we wanted to view the article, we would make a `GET` request to `/articles/4`. But what about when I want to update that article? Am I hitting a different resource? Nope! Just doing a different action to that same resource. So instead of a `GET` against `/articles/4` we do a `PUT`. That's why separating what you're talking to (the resource/noun) from the action you're doing (the HTTP verb) is important! That's key to REST.

## Browser Caveat

Browsers behave a little strangely as it relates to `PUT`, `PATCH` and `DELETE` requests, in that they don't know how to send those requests. Forms that delete and edit need to be submitted via `POST` requests.

## Routes Overview

Let's take a magazine website as an example. You'd want to have a controller action to create a new article (new route), to display one article (show route), to display all articles (index route), to delete an article (delete route), and a page to edit an article (edit route).

| HTTP VERB | ROUTE              | Action        | Used For                                               |
|---      |---                 |---            |---                                                     |
| GET     | '/articles'           | index action  | index page to display all articles                        |
| GET     | '/articles/new'       | new action    | displays create article form                              |
| POST    | '/articles'           | create action | creates one article                                  |
| GET     | '/articles/:id'       | show action   | displays one article based on ID in the url          |
| GET     | '/articles/:id/edit'  | edit action   | displays edit form based on ID in the url              |
| PATCH   | '/articles/:id'       | update action | _modifies_ an existing article based on ID in the url|
| PUT     | '/articles/:id'       | update action | _replaces_ an existing article based on ID in the url|
| DELETE  | '/articles/:id'       | delete action | deletes one article based on ID in the url           |

## The Routes

###  Index Action

```ruby
get '/articles' do
  @articles = Article.all
  erb :index
end
```

The controller action above responds to a `GET` request to the route `'/articles'`. This action is the `index action` and allows the view to access all the articles in the database through the instance variable `@articles`.


### New Action

```ruby
get '/articles/new' do
  erb :new
end

post '/articles' do
  @article = Article.create(:title => params[:title], :content => params[:content])
  redirect to "/articles/#{@article.id}"
end
```

Above, we have two controller actions. The first one is a `GET` request to load the form to create a new article. The second action is the `create action`. This action responds to a `POST` request and creates a new article based on the params from the form and saves it to the database. Once the item is created, this action redirects to the `show` page.


### Show Action

```ruby
get '/articles/:id' do
  @article = Article.find_by_id(params[:id])
  erb :show
end
```

In order to display a single article, we need a `show action`. This controller action responds to a `GET` request to the route `'/articles/:id'`. Because this route uses a dynamic URL, we can access the ID of the article in the view through the `params` hash.

### Edit Action

```ruby
get '/articles/:id/edit' do  #load edit form
    @article = Article.find_by_id(params[:id])
    erb :edit
  end

patch '/articles/:id' do #edit action
  @article = Article.find_by_id(params[:id])
  @article.title = params[:title]
  @article.content = params[:content]
  @article.save
  redirect to "/articles/#{@article.id}"
end
```

The first controller action above loads the edit form in the browser by making a `GET` request to `articles/:id/edit`.

The second controller action handles the edit form submission. This action responds to a `PATCH` request to the route `/articles/:id`. First, we pull the article by the ID from the URL, then we update the title and content attributes and save. The action ends with a redirect to the article show page.

We do have to do a little extra work to get the edit form to submit via a `PATCH` request.

Your form must include a hidden input field that will submit our form via `patch`.

```html
<form action="/articles/<%= @article.id %>" method="post">
  <input id="hidden" type="hidden" name="_method" value="patch">
  <input type="text" name="title">
  <input type="text" name="content">
  <input type="submit" value="submit">
</form>
```

The second line above `<input type="hidden" name="_method" value="patch">` is what does this for us.

#### Using `PATCH`, `PUT` and `DELETE` requests with `Rack::MethodOverride` Middleware

The hidden input field shown above uses `Rack::MethodOverride`, which is part of [Sinatra middleware](https://github.com/rack/rack/blob/master/lib/rack/method_override.rb). 

In order to use this middleware, and therefore use `PATCH`, `PUT`, and `DELETE` requests, you *must* tell your app to use the middleware.

In the `config.ru` file, you'll need the following line to be placed *above* the `run ApplicationController` line:

```ruby
use Rack::MethodOverride
```

In an application with multiple controllers, `use Rack::MethodOverride` must be placed **above all controllers** in which you want access to the middleware's functionality.

This middleware will then run for every request sent by our application. It will interpret any requests with `name="_method"` by translating the request to whatever is set by the `value` attribute. In this example, the `post` gets translated to a `patch` request. The middleware handles `put` and `delete` in the same way.

Many developers are confused about the difference between `PATCH` and `PUT`.  Imagine a car with a license plate (`id`). Now let's say we wanted to change the car's color from red to green. We could:

1. Pull our our disintegrating raygun and zap the car **ZZZZAP** and build a _new_ car that was identical to the first car in all aspects except that it was green instead of red. We could slap the old license plate (`id`) on it and, from a certain point of view, we have "updated the Car with given license plate with `id` equal to `params[:id]`
2. Find a given car and repaint it

Option 1 is like `PUT` a replace of all fields. Option 2 is like a `PATCH`. The subtler question of what differentiates the two hinges on a fancy Latin-esque word: _idempotent_. If you're really curious about the subtleties here, check out this [Stack Overflow](https://stackoverflow.com/questions/28459418/rest-api-put-vs-patch-with-real-life-examples) question. It may suffice to say that `PATCH` is relatively new and in the early days of REST we only used `PUT` (We were zapping all day long
!).

### Delete Action

```ruby
delete '/articles/:id' do #delete action
  @article = Article.find_by_id(params[:id])
  @article.delete
  redirect to '/articles'
end
```

On the article show page, we have a form to delete it. The form is submitted via a `DELETE` request to the route `/articles/:id`. This action finds the article in the database based on the ID in the url parameters, and deletes it. It then redirects to the index page `/articles`.

Again, this delete form needs the hidden input field:

```html
<form action="/articles/<%= @article.id %>" method="post">
  <input id="hidden" type="hidden" name="_method" value="delete">
  <input type="submit" value="delete">
</form>
```

<p class='util--hide'>View <a href='https://learn.co/lessons/sinatra-restful-routes-readme'>Sinatra RESTful Routes</a> on Learn.co and start learning to code for free.</p>
