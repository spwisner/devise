# Devise

## Learning Objectives

- Implement user authentication
- `merge` two hashes to associate the user model
- Use Devise helper methods
- Explain the difference between encryption and hashing algorithms

## Setup (5 minutes / 0:05)

```
$ git clone https://github.com/ga-wdi-exercises/scribble.git
$ cd scribble
$ git checkout pre-devise
$ rails db:drop db:create db:migrate db:seed
$ rails s
```

## Devise (5 minutes / 0:10)

[Devise](https://github.com/plataformatec/devise#starting-with-rails) is a gem that simplifies implementing user authentication.

While it's interesting to know what's happening under the hood when authenticating a user, it's very unlikely you'll implement your own authentication mechanism in any future project. Devise is a well-tested open source authentication solution for rails.

In yesterday's lesson, passwords were stored in the database in plaintext.

<details>
  <summary><strong>Why might storing passwords in the database be a bad idea?</strong></summary>

  - We might get hacked
  - Some users have the same password for every service
  - [Plain Text Offenders](http://plaintextoffenders.com/faq/devs)

</details>

<details>
  <summary><strong>What are the alternatives to storing passwords in plaintext?</strong></summary>

  - Hashing
  - Encryption

</details>

## You Do: Stack Overflow Hunt (10 minutes / 0:20)

> 5 minutes exercise. 5 minutes review.

Read through the question and answers in [this Stack Overflow post](http://www.securityinnovationeurope.com/blog/whats-the-difference-between-hashing-and-encrypting).

Prepare to answer the following questions...

<details>
  <summary><strong>What is the difference between encryption and hashing?</strong></summary>

  > **Encryption** takes a string and, using a key/algorithm, converts it to another string of variable length. The key/algorithm can be used to revert -- or "decrypt" -- the new string back to the original. In other words, it is a two-way process.

</details>

<details>
  <summary><strong>What is the difference between encryption and hashing?</strong></summary>

  > **Encryption** takes a string and, using a key/algorithm, converts it to another string of variable length. The key/algorithm can be used to revert -- or "decrypt" -- the new string back to the original. In other words, it is a two-way process.
  >
  > **Hashing** is similar to encryption in that it converts a string to a different one using an algorithm. It cannot, however, be reverted back to the original, making it a one-way process.

</details>

<details>
  <summary><strong>When would you choose one over the other?</strong></summary>

  > Encryption should only be used if somebody or something needs to see the original string.

</details>

<details>
  <summary><strong>Which one should be used for passwords?</strong></summary>

  > **Hashing**. For security reasons, there is no need to store or gain access to the original password. All an application needs to do is to take in a password attempt, pass it through the hashing algorithm and see if it matches with the hash generated when the particular user signed for an account with an app.

</details>

## Installing Devise (5 minutes / 0:25)

In order to use Devise, we must include it in our Gemfile...

```rb
# Gemfile

gem "devise"
```

```bash
$ bundle install
$ rails g devise:install
```

Of the instructions listed in the resulting output, we only need to do the third item.

```
 3. Ensure you have flash messages in app/views/layouts/application.html.erb.

For example:

<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

<details>
  <summary><strong>When working with Devise, what do you think might show up inside a flash message?</strong></summary>

  - Whether the user has signed in or out successfully
  - Whether the password is correct/incorrect
  - Whether the username is taken

</details>

### Commit!

Seriously, commit now. It will be easy to fix any devise issues if you have a commit you can go back to.

### You Do: Generate User Model (10 minutes / 0:35)

> 5 minutes exercise. 5 minutes review.

In the User, Sessions & Flash lesson, you set up a User model manually. When using Devise, you should take advantage of its built in model generation functionality.

Run this in the command line...

```bash
$ rails g devise User
```

> If you have an existing User model, Devise should build on the functionality. But, if you're getting migration errors, it may be easier to recreate the model using Devise and add your functionality back.

Once you've done that, look through your application and answer the following questions...
- What did the previous command generate?
- What files did running the previous command change?
- What is the output of `rails routes`?

### Run Migrations

The above command generated some model and migration files. It did not, however, run said migrations. Let's do that now.

```bash
$ rails db:migrate
$ rails s
```

### Link to "Sign Up" Page (5 minutes / 0:40)

While Devise does a lot of leg work for us, we need to go into our views and make sure we provide users with the option to create a Scribble account.

Let's add a link to do that in `app/views/layouts/application.html.erb`...

```erb
<!-- app/views/layouts/application.html.erb -->
<%= link_to "Sign Up", new_user_registration_path %>
```

> Where did that `new_user_registration_path` path helper come from? Devise.
>
> If you want to look at what other path helpers Devise has generated for us, run `rails routes` in the Terminal.

### Link to "Sign Up" Only If Not Signed In (5 minutes / 0:45)

We don't want a user to see the "Sign Up" link if they're already signed in. What logic can we implement in our layout file to prevent this from happening?

```erb
<!-- app/views/layouts/application.html.erb -->
<% if !current_user %>
  <%= link_to "Sign Up", new_user_registration_path %>
<% end %>
```
> `current_user` is a helper method provided by Devise to get the user from the session. Another helper method you can use to check if a user is signed in is the self-explanatory user_signed_in?.

### You Do: "Log Out" Link (15 minutes / 1:00)

> 10 minutes exercise. 5 minutes review.

If the user is logged in, show a link to log out. The link should display the user's email right next to it.

Otherwise, show both a link to sign up and sign in.

> Where should we be directing the "Log Out" link? `rails routes` will be very helpful in answering that question.

<details>
  <summary><strong>Solution...</strong></summary>

  ```erb
  <!-- app/views/layouts/application.html.erb -->
  <% if !current_user %>
    <%= link_to "Sign up", new_user_registration_path %>
    <%= link_to "Sign in", new_user_session_path %>
  <% else %>
    <%= link_to "Sign out", destroy_user_session_path, :method => :delete %>
      <%= current_user.email %>
  <% end %>
  ```

  > The delete method needs to be specified. `GET` is the default for the `link_to` helper.

</details>

### Break (10 minutes / 1:10)

### You Do: Associate Posts with a User (10 minutes / 1:20)

> 5 minutes exercise. 5 minutes review.

Modify your `User` and `Post` classes so that...
- A user has many posts
- A post belongs to a user.

Create a few seeds to verify you did this part correctly.

<details>
  <summary><strong>Solution...</strong></summary>

  ```rb
  # app/models/user.rb

  has_many :posts
  ```

  ```rb
  # app/models/post.rb

  belongs_to :user
  ```

  ```
  $ rails g migration add_users_to_posts user:references
  $ rails db:migrate
  ```

</details>

### We Do: Update the Controller (10 minutes / 1:30)

Now let's make use of these associations. In particular, let's make it so that when a user creates a post, that post is automatically associated with that user.

There are a couple ways of doing this.

One is to leverage the `current_user` method we encountered before.

```rb
# app/controllers/posts_controller.rb

def create
  current_user.posts.create!(post_params)
  redirect_to post_path(@post)
end
```
> Remember: `current_user` returns an instance of the logged-in user.

The other is to make use of Ruby's `[.merge](https://ruby-doc.org/core-2.2.0/Hash.html#method-i-merge)` method, which combines two hashes into one.

```rb
# app/controllers/posts_controller.rb

def create
  @post = Post.create!(post_params.merge(user: current_user))
  redirect_to post_path(@post)
end
```
> `post_params` and `{user: current_user}` are both hashes. Together, they contain all of the information we would like a post to have.

Note that we did not have to modify strong params in either option.

### Limiting User Abilities (5 minutes / 1:35)

Right now our users have free reign over Scribble. Let's limit their permissions. First, we'll prevent a user from being able to delete somebody else's post.

First let's add some logic to our posts controller's `destroy` action so that, if a user tries to delete a different user's post, they see an error flash message.

```rb
# app/controllers/posts_controller.rb

def destroy
  @post = Post.find(params[:id])
  if @post.user == current_user
    @post.destroy
  else
    flash[:alert] = "Only the author of the post can delete"
  end
  redirect_to posts_path
end
```

This is nice, but it'd be better if they didn't see the delete link in the first place...

### You Do: Prevent Users From Deleting & Editing Someone Else's Post (15 minutes / 1:50)

> 10 minutes exercise. 5 minutes review.

Start by making it so that a user cannot see the "Delete" link for a post they did not create themselves.

Now do the same thing with edit functionality, making sure to modify the appropriate controller(s) and view(s).

<details>
  <summary><strong>Solution...</strong></summary>

  ```rb
  def edit
    @post = Post.find(params[:id])
    if @post.user == current_user
      @post.destroy
    else
      flash[:alert] = "Only the author of the post can delete"
    end
    redirect_to posts_path
  end
  ```

</details>

### Break (10 minutes / 2:00)

### You Do: Associate Users With Comments (20 minutes / 2:20)

> 15 minutes exercise. 5 minutes review.

1. Create a new migration to add a user_id column to comments
- Associate the `current_user` with newly created posts
- Prevent User's from editing / deleting other's comments

## Closing / Questions

## Want More Practice?

Implement Devise in Scribble. Make sure, however, to work on it on a separate branch and merge it to `master` when you're ready.

------------

## Customizing Views

Devise has generated several views for us but they are not currently visible in our
codebase.

If you want to customize them, generate the views with `rails g devise:views`

### Styling Views

Whether you generated the views or not, you can style the forms the same way.

Identify the selectors you would use to target the individual form elements, and add
styles in `app/assets/stylesheets/application.css`

## If There's Time...

Try implementing an authorization solution on top of Devise using [CanCanCan](https://github.com/ga-wdi-lessons/cancancan)
