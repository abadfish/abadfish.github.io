---
published: true
title: Rails App With A JQuery Front-End
layout: post
author: Kirsten O'Farrell
---
Using JQuery To Display HTML
-------------------------------

So we know that every time the user makes a request for data to be displayed an HTML request is made to the server. Without Javascript, this is a full page re-load asking that the server display everything that was there before the request, in addition to the new data. With Javascript, however, we can just ask the server for the new info requested. Let's walk through an example:

Say we have 2 models in our app, authors and their posts. We would like to go to an author's show page and be able to press a button to show a list of that author's posts. For this we're going to use the gem ```
active_model_serializers
```
Add it and run bundle. Then run
```
rails g serializer author
```
in your console.

The ActiveRecord relationship is such that a post belongs to an author and an author has many posts. In your newly generated AuthorSerializer add the following:

```
class AuthorSerializer < ActiveModel::Serializer
  attributes :id, :name
  has_many :posts
end
```

Next go over to your authors controller. We have to write an action to display the posts associated with the author in .json form so we can make an AJAX call to display it on the author page.

```
def author_data
  author = Author.find(params[:id])
  render json: author.to_json(only: [:name, :hometown, :id], include: [posts: {only: [:title, :content]}])
end
```

Then go to your routes.rb file and add a route to this controller action:

```
get 'authors/:id/author_data', to: 'authors#author_data'
```

Lastly, we have to add the script to the author's show page to display the actual data.
Assuming you already have the jquery gem in your gemfile and jquery in your asset pipeline (if you do not, add them now), add some script tags to your file and add a button. The press of the button will be the event used to call the author_data so lets add a class to the button in order to make a connection to the script. Also add a data-id to associate the script with the specific author who's page we are on.

```
<button class="js-show btn" data-id="<%= @author.id %>">See Posts By <%= @author.name %></button>
```

We know because of the relationship that we can iterate through an author's posts with @author.posts.

```<% @author.posts.each do |post| %>
    <h4 id="postTitle"></h4>
    <p id="postContent"></p>
  <% end %>
```

The id's here associated with the HTML tags are what we will use to connect the script to the data for display. Here we are telling the DOM where to put the data and how to display it.

>
```
<script type="text/javascript" charset="utf-8">
$(function () {
  $(".js-show").on('click', function() {
    var id = $(this).data("id");
    $.get("/authors/" + id + "/author_data", function(data) {
      var posts = data["posts"];
      for (var i = 0; i < posts.length; i++){
      var title = posts[i]["title"];
      var content = posts[i]["content"];
    };
      $("#postTitle").html(title);
      $("#postContent").html(content);
    });
  });
});
</script>
```

So here we go: $ means we have a jQuery object, $( means we're calling a function on it.
$(".js-show") is the event selector. When we "click" the button with the class "js-show" the jQuery runs the function written directly after it. We tell it that the id is based on the author's id. We defined this with data-id and we define it in the script with

```
var id = $(this).data("id");
```

Then we make the actual AJAX get request using the author_data url for the author.
We assign variables to the data we get back and then tell them where to display using the id's we defined in the HTML tags above.

And there you have it. You are displaying the posts of an author without having to reload the page.


A working example of the code written above can be seen in this video: <a href="https://youtu.be/KKmST27BgLc">Rails App with JQuery Front-End</a>.

If you'd like to take a look at the code or offer suggestions please do so on <a href="https://github.com/abadfish/rails-jquery-assessment">GitHub.</a>
