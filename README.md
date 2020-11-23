# RESTful Routes

## Overview

In this lesson we'll explain the benefits of RESTful routes and how they provide a design pattern that allows for easy data manipulation.

## Objectives

+ Explain the concept of RESTful routes
+ Implement RESTful routes in a Sinatra application

## What Is A RESTful Route?

The internet would be a really confusing place without a convention for how to
handle URLs &mdash; to delete an Instagram photo might be
`www.instagram.com/delete-this-photo`, but Twitter might be
`www.twitter.com/remove-this-tweet`. Without a specific convention to follow, it
would be harder to create new content, edit content, and delete it. REST (which
stands for REpresentational State Transfer) provides a consistent pattern to use
in structuring routes. Following RESTful patterns makes it easier for developers
to create and maintain routes and easier for users to understand what's
happening as they use a web application.

A RESTful route is a route that provides mapping from HTTP verbs (get, post,
put, delete, patch) to controller CRUD actions (create, read, update, delete).
Instead of relying solely on the URL to indicate what site to visit, a RESTful
route depends on the HTTP verb __and__ the URL.

What this means is that when your application receives an HTTP request, it
identifies the HTTP method and URL contained in the request, finds the
controller action that consists of that method and URL, executes the code in
that constroller action, and determines which response gets sent back to the
client. We don't need to worry about how the mechanics of the pattern matching
occurs, just that it does happen.

It's important to remember that CRUD actions are identified by a combination of
an HTTP verb and a resource (identified by the URL), so there will be different
actions that occur on the same resource. Let's take the example of an article
with the ID 4. If we wanted to view the article, we would make a `GET` request
to `/articles/4`. But what about when I want to update that article? Am I
hitting a different resource? Nope! Just doing a different action to that same
resource. So instead of a `GET` against `/articles/4` we do a `PUT` or `PATCH`
to that same URL. That's why separating what you're talking to (the
resource/noun) from the action you're doing (the HTTP verb) is important! That's
key to REST.

## Browser Caveat

Browsers behave a little strangely as it relates to `PUT`, `PATCH` and `DELETE`
requests, in that they don't know how to send those requests. Forms that delete
and edit need to be submitted via `POST` requests. We will learn a little later
in this lesson how to use the `Rack::MethodOverride` Middleware to send `PUT`,
`PATCH` and `DELETE` requests.

## Routes Overview

Let's take a magazine website as an example. You'd want to have a controller
action to create a new article (new route), to display one article (show route),
to display all articles (index route), to delete an article (delete route), and
to edit an article (edit route).

<table border="1" cellpadding="4" cellspacing="0">
  <tr>
    <th>HTTP Verb</th>
    <th>Route</th>
    <th>Action</th>
    <th>Used For</th>
  </tr>
  <tr>
    <td>GET</td>
    <td>'/articles'</td>
    <td>index action</td>
    <td>index page to display all articles</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>'/articles/new'</td>
    <td>new action</td>
    <td>displays create article form</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>'/articles'</td>
    <td>create action</td>
    <td>creates one article</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>'/articles/:id'</td>
    <td>show action</td>
    <td>displays one article based on ID in the url</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>'/articles/:id/edit'</td>
    <td>edit action</td>
    <td>displays edit form based on ID in the url</td>
  </tr>
  <tr>
    <td>PATCH</td>
    <td>'/articles/:id'</td>
    <td>update action</td>
    <td><em>modifies</em> an existing article based on ID in the url</td>
  </tr>
  <tr>
    <td>PUT</td>
    <td>'/articles/:id'</td>
    <td>update action</td>
    <td><em>replaces</em> an existing article based on ID in the url</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>'/articles/:id'</td>
    <td>delete action</td>
    <td>deletes one article based on ID in the url</td>
  </tr>
</table>

## The Routes

### Index Action

```ruby
get '/articles' do
  @articles = Article.all
  erb :index
end
```

The controller action above responds to a `GET` request to the route
`'/articles'`. This action is the `index action` and allows the view to display
all the articles in the database using the instance variable `@articles`.

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

Above, we have two controller actions. The first one is a `GET` request to load
the form to create a new article. The second action is the `create action`. This
action responds to a `POST` request. When the `new` form is submitted, it
creates a new article based on the params from the form and saves it to the
database. Once the item is created, this action redirects to the `show` page.

### Show Action

```ruby
get '/articles/:id' do
  @article = Article.find_by_id(params[:id])
  erb :show
end
```

In order to display a single article, we need a `show action`. This controller
action responds to a `GET` request to the route `'/articles/:id'`. Because this
route uses a dynamic URL, we can access the ID of the article through the
`params` hash, retrieve the appropriate article from the database, and pass it
to the show view using the `@article` instance variable.

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

The first controller action above loads the edit form for a specific article in
the browser by making a `GET` request to `articles/:id/edit`.

The second controller action handles the edit form submission. This action
responds to a `PATCH` request to the route `/articles/:id`. First, we pull the
article by the ID from the URL, then we update the title and content attributes
and save. The action ends with a redirect to the article show page.

Recall that we mentioned earlier in this lesson that we have to do a little
extra work to submit `PUT`, `PATCH` and `DELETE` requests because browsers don't
recognize these verbs. To get the edit form to submit via a `patch` request,
your form must include a hidden input field:

```html
<form action="/articles/<%= @article.id %>" method="post">
  <input id="hidden" type="hidden" name="_method" value="patch">
  <input type="text" name="title">
  <input type="text" name="content">
  <input type="submit" value="submit">
</form>
```

The second line above `<input type="hidden" name="_method" value="patch">` is
what does this for us.

#### Using `PATCH`, `PUT` and `DELETE` requests with `Rack::MethodOverride` Middleware

The hidden input field shown above uses `Rack::MethodOverride`, which is part of [Sinatra middleware](https://github.com/rack/rack/blob/master/lib/rack/method_override.rb).

In order to use this middleware, and therefore use `PATCH`, `PUT`, and `DELETE`
requests, you *must* tell your app to use the middleware.

In the `config.ru` file, you'll need the following line to be placed *above* the
`run ApplicationController` line:

```ruby
use Rack::MethodOverride
```

In an application with multiple controllers, `use Rack::MethodOverride` must be
placed **above all controllers** in which you want access to the middleware's
functionality.

This middleware will then run for every request sent by our application. It will
use the `name` and `value` attributes in the hidden field to translate the
request. Specifically, in the example above, `name="_method"` tells the
middleware to translate the form tag's `method` attribute and `value="patch"`
tells it to change the `method`s `value` to `patch`. The middleware handles
`put` and `delete` in the same way.

#### `PATCH` VS. `PUT`

Many developers are confused about the difference between `PATCH` and `PUT`.
Imagine a car with a license plate (`id`). Now let's say we wanted to change the
car's color from red to green. We could:

1. Pull out our disintegrating raygun and zap the car **ZZZZAP** and build a
   _new_ car that was identical to the first car in all aspects except that it
   was green instead of red. We could slap the old license plate (`id`) on it
   and, from a certain point of view, we have "updated the Car with `id`
   (license plate) equal to `params[:id]`"
2. Find a given car and repaint it

Option 1 is like `PUT` a replace of all fields. Option 2 is like a `PATCH`. The subtler question of what differentiates the two hinges on a fancy Latin-esque word: _idempotent_. If you're really curious about the subtleties here, check out this [Stack Overflow](https://stackoverflow.com/questions/28459418/rest-api-put-vs-patch-with-real-life-examples) question. It may suffice to say that `PATCH` is relatively new and in the early days of REST we only used `PUT` (we were zapping all day long!).

### Delete Action

```ruby
delete '/articles/:id' do #delete action
  @article = Article.find_by_id(params[:id])
  @article.delete
  redirect to '/articles'
end
```

On the article show page, we have a form to delete it. The form is submitted via
a `DELETE` request to the route `/articles/:id`. This action finds the article
in the database based on the ID in the url parameters and deletes it. It then
redirects to the index page `/articles`.

Again, this delete form needs the hidden input field:

```html
<form action="/articles/<%= @article.id %>" method="post">
  <input id="hidden" type="hidden" name="_method" value="delete">
  <input type="submit" value="delete">
</form>
```
