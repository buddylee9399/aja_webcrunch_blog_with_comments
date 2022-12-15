# Web crunch - blog
- 	⁃	web crunch stripe tutorial
- rails g scaffold post title content:text
- rails g scaffold comment name comment:text
- rails g migration add_post_id_to_comments post_id:integer
- update routes

```
Rails.application.routes.draw do
  resources :posts do 
    resources :comments
  end
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Defines the root path route ("/")
  # root "articles#index"
  root 'posts#index'
end

```

- change app.scss and add bulma

```
@import "bulma";
```

- update comments controller

```
class CommentsController < ApplicationController

  def create
    @post = Post.find(params[:post_id])
    @comment = @post.comments.create(params[:comment].permit(:name, :comment))
    redirect_to post_path(@post)  
  end

  def destroy
    @post = Post.find(params[:post_id])
    @comment = @post.comments.find(params[:id])
    @comment.destroy
    redirect_to post_path(@post)
  end
end
```

- update comment.rb

```
class Comment < ApplicationRecord
	belongs_to :post
end

```

- update post.rb

```
class Post < ApplicationRecord
	has_many :comments, dependent: :destroy
end
```

- rails g simple_form:install
- update comment form partial

```

<%= simple_form_for([@post, @post.comments.build]) do |f| %>
<!--
collection.build(attributes = {}, …) Returns one or more new objects of the collection type that have been instantiated with attributes and linked to this object through a foreign key, but have not yet been saved. Note: This only works if an associated object already exists, not if it‘s nil!
-->
<div class="field">
  <div class="control">
    <%= f.input :name, input_html: { class: 'input' }, wrapper: false, label_html: { class: 'label' } %>
  </div>
</div>

<div class="field">
  <div class="control">
    <%= f.input :comment, input_html: { class: 'textarea' }, wrapper: false, label_html: { class: 'label' }  %>
  </div>
</div>
<%= f.button :submit, 'Leave a reply', class: "button is-primary" %>
<% end %>
```

- update comment comment partial

```
<div class="box">
  <article class="media">
    <div class="media-content">
      <div class="content">
        <p>
          <strong><%= comment.name %>:</strong>
          <%= comment.comment %>
        </p>
      </div>
    </div>
     <%= link_to 'Delete', [comment.post, comment],
                  method: :delete, class: "button is-danger", data: { confirm: 'Are you sure?' } %>
  </article>

</div>
```

- update layout app html file

```
<!DOCTYPE html>
<html>
  <head>
    <title>AjaWebcrunchBlogWithComments</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <section class="hero is-primary is-medium">
    <!-- Hero head: will stick at the top -->
    <div class="hero-head">
      <nav class="navbar">
        <div class="container">
          <div class="navbar-brand">
            <%= link_to 'Demo Blog', root_path, class: "navbar-item" %>
            <span class="navbar-burger burger" data-target="navbarMenuHeroA">
              <span></span>
              <span></span>
              <span></span>
            </span>
          </div>
          <div id="navbarMenuHeroA" class="navbar-menu">
            <div class="navbar-end">
              <%= link_to "Create New Post", new_post_path, class:"navbar-item" %>
            </div>
          </div>
        </div>
      </nav>
    </div>

    <!-- Hero content: will be in the middle -->
    <div class="hero-body">
      <div class="container has-text-centered">
        <h1 class="title">
          <%= yield :page_title %>
        </h1>
      </div>
    </div>
  </section>
    <%= yield %>
  </body>
</html>
```

- update post form partial

```
<div class="section">
<%= simple_form_for @post do |f| %>
  <div class="field">
    <div class="control">
      <%= f.input :title, input_html: { class: 'input' }, wrapper: false, label_html: { class: 'label' } %>
    </div>
  </div>

  <div class="field">
    <div class="control">
      <%= f.input :content, input_html: { class: 'textarea' }, wrapper: false, label_html: { class: 'label' }  %>
    </div>
  </div>
  <%= f.button :submit, 'Create new post', class: "button is-primary" %>
<% end %>
</div>
```

- update post edit

```
<% content_for :page_title, "Edit Post" %>

<%= render "form", post: @post %>
```

- update post new

```
<% content_for :page_title, "Create a new post" %>

<%= render "form", post: @post %>

```
- update post index

```

<% content_for :page_title,  "Index" %>

<div class="section">
  <div class="container">
    <% @posts.each do |post| %>
      <div class="card">
      <div class="card-content">
        <div class="media">
          <div class="media-content">
            <p class="title is-4"><%= link_to post.title, post  %></p>
          </div>
        </div>
        <div class="content">
          <%= post.content %>
        </div>
        <div class="comment-count">
          <span class="tag is-rounded"><%= post.comments.count %> comments</span>
        </div>
      </div>
    </div>
    <% end %>
  </div>
</div>
```

- update post show

```
<% content_for :page_title, @post.title %>

<section class="section">
	<div class="container">
		<nav class="level">
		  <!-- Left side -->
		  <div class="level-left">
		    <p class="level-item">
		        <strong>Actions</strong>
		    </p>
		  </div>
		  <!-- Right side -->
		  <div class="level-right">
		  	<p class="level-item">
		    	<%= link_to "Edit", edit_post_path(@post), class:"button" %>
		  	</p>
		  	<p class="level-item">
				<%= link_to "Delete", post_path(@post), method: :delete, data: { confirm: "Are you sure?" }, class:"button is-danger" %>
				</p>
		  </div>
		</nav>
		<hr/>

		<div class="content">
			<%= @post.content %>
		</div>
	</div>
</section>

<section class="section comments">
	<div class="container">
		<h2 class="subtitle is-5"><strong><%= @post.comments.count %></strong> Comments</h2>
		<%= render @post.comments %>
		<div class="comment-form">
			<hr />
			<h3 class="subtitle is-3">Leave a reply</h3>
	 		<%= render 'comments/form' %>
		</div>
	</div>
</section>
```

- launch and test out

## THE END