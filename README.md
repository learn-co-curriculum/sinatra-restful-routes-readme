# RESTful Routes 

## Objectives
+ Explain the concept of RESTful routes

## What Is A RESTful Route?

What is the internet without a convention for how to handle URLS - to delete a Facebook post might be www.facebook.com/delete-this-wallpost, but Twitter might be www.twitter.com/remove-this-tweet. Without a specific convention to follow, it would be hard to create new content, edit content, and delete it. RESTful routes provides a design pattern that allows for easy data manipulation. It's nicer for users and nicer for developers to have everything consistent.

A RESTful route is a route that provides mapping between HTTP verbs (get post, put, delete, patch) to controller CRUD actions (create, read, update, delete). Instead of relying solely on the URL to indicate what site to visit, a RESTful route also depends on the HTTP verb __and__ the URL. 

What this means is that when your application receives an HTTP request, it introspects on that request and identifies the HTTP method and URL,connects that with a corresponding controller action that has that method and URL, executes the code in that action, and determines which response gets sent back to the client. We don't need to worry about how the mechanics of the pattern matching occurs, just that it does happen.


## The Routes

Let's take a blog website as an example. You'd want to have a controller action to create a new post (new route), to display one post (show route), to display all posts (index route), to delete a post (delete route), and a page to edit a post (edit route).

| HTTP VERB | ROUTE  |  Action |   Used For|  # |
|---|---|---|---|---|
|  GET |  '/posts' | index action   | index page to display all posts   |   |
|   |   |   |   |   |
|   |   |   |   |   |

## chart like rails

## explanation of each

they are stateless - meaning each transaction is unique and unrelated to previous requests
GET requests to load forms ('/posts/:id/edit' to load edit form, '/posts/new' to load new form)
get '/posts' do # (index action, displays all posts)
get '/posts/:id' (show action, displays one post based on ID in URL)
patch '/posts/:id' (edit acton, edits one post based on ID in URL)
delete 'posts/:id' (delete action, deletes one post paste on ID in URL)
post 'post/new' (new/create action, creates one post and saves to DB)
 THIS IS BASED OFF ONE APPLICATION CONTROLLER