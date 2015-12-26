# RESTful Routes 

## Objectives
+ Explain the concept of RESTful routes
+ Implement RESTful routes in a Sinatra application

## What Is A RESTful Route?

The internet would be a really confusing place without a convention for how to handle URLS - to delete a Facebook post might be www.facebook.com/delete-this-wallpost, but Twitter might be www.twitter.com/remove-this-tweet. Without a specific convention to follow, it would be hard to create new content, edit content, and delete it. RESTful routes provides a design pattern that allows for easy data manipulation. It's nicer for users and nicer for developers to have everything consistent.

A RESTful route is a route that provides mapping between HTTP verbs (get post, put, delete, patch) to controller CRUD actions (create, read, update, delete). Instead of relying solely on the URL to indicate what site to visit, a RESTful route also depends on the HTTP verb __and__ the URL.

What this means is that when your application receives an HTTP request, it introspects on that request and identifies the HTTP method and URL,connects that with a corresponding controller action that has that method and URL, executes the code in that action, and determines which response gets sent back to the client. We don't need to worry about how the mechanics of the pattern matching occurs, just that it does happen.

It's important to note that much of the CRUD actions are different actions that occur on the same resource. Let's take the example of a blog post with the ID 4. If we wanted to view the post, we would make a `GET` request to `/posts/4`. But what about when I want to update that post? Am I hitting a different resource? Nope! Just doing a different action to that same resource. So instead of a `GET` against `post/4` we do a `PUT`. That's what separating what you're talking to (the resource/noun) from the action you're doing (the HTTP verb) is important! That's key to REST.

## Browser Caveat

Browers behaves a little strange as it relates to `PUT`, `PATCH` and `DELETE` requests, in that they don't know how to send to those requests. Forms to delete and edit need to be submitted via `POST` requests.

## Routes Overview

Let's take a blog website as an example. You'd want to have a controller action to create a new post (new route), to display one post (show route), to display all posts (index route), to delete a post (delete route), and a page to edit a post (edit route).

| HTTP VERB | ROUTE | Action | Used For |
|---        |---    |---      |---      |
|  GET |  '/posts' | index action   | index page to display all posts   |
|   GET |   '/posts/:id'| show action   |displays one blog post based on ID in the url|
|   PATCH (Sinatra POST)| '/posts/:id/edit'   | edit action   | edits one blog post based on ID in the url  |
|   DELETE (Sinatra POST)| '/posts/:id/delete'   | delete action   |deletes one blog post based on ID in the url  |
|   POST| '/posts'   | create action   |creates one blog post |


## The Routes

###  Index Action

```ruby
get '/posts' do
  @posts = Post.all
  erb :index
end
```

The controller action above responds to a `GET` request to the route `'/posts'`. This action is the `index action` and allows the view to access all the posts in the database through the instance variable `@posts`.


### New Action

```ruby
get '/posts/new' do
  erb :new
end

post '/posts' do
  @post = Post.create(:title => params[:title], :content => params[:content])
  redirect to '/posts/#{@post.id}'
end
```

Above, we have two controller actions. The first one is a `GET` request to load the form to create a new blog post. The second action is the `create action`. This action responds to a `POST` request and creates a new post based on the params from the form and saves it to the database. Once the item is created, this action redirects to the `show` page. 


### Show Action

```ruby
get '/posts/:id'
  @post = Post.find_by_id(params[:id])
  erb :show
end
```

In order to display a single post, we need a `show action`. This controller action responds to a `GET` request to the route `'/posts/:id'`. Because route uses a dynamic URL, we can access the ID of the post in the view through the `params` hash.

### Edit Action

```ruby
get '/posts/:id/edit' do  #load edit form
    @post = Post.find_by_id(params[:id])
    erb :edit
  end

post '/posts/:id' do #edit action
  @post = Post.find_by_id(params[:id])
  @post.title = params[:title]
  @post.content = params[:content]
  @post.save
  redirect to "/posts/#{@post.id}"
end
```

The first controller action above loads the edit form in the browser by making a `GET` request to `posts/:id/edit`.

The second controller action handles the edit form submission. This action responds to a `POST` request (because Sinatra can't handle `PUT`) to the route `/posts/:id`. First, we pull the blog post by the ID from the URl, then we update the title and content attributes and save. The action ends with a redirect to the blog post show page.


### Delete Action

```ruby
post '/posts/:id/delete' do #delete action
  @post = Post.find_by_id(params[:id])
  @post.delete
  redirect to '/posts'
end
```

On the blog post show page, we have a form to delete it. The form is submitted via a `POST` request (again, because Sinatra can't handle `DELETE` requests) to the route `/posts/:id/delete`. This action finds the blog post in the database based on the ID in the url parameters, and deletes it. It then redirects to the index page `/posts`.



<a href='https://learn.co/lessons/sinatra-restful-routes-readme' data-visibility='hidden'>View this lesson on Learn.co</a>

