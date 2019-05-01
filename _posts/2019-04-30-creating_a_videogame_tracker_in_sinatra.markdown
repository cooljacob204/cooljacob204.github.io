---
layout: post
title:      "Creating a Videogame Tracker in Sinatra"
date:       2019-04-30 20:09:50 -0400
permalink:  creating_a_videogame_tracker_in_sinatra
---


## https://videogame-tracker.lfp2.gg

## Getting started on my app

Coming up with an idea for a project isn't always easy. This is a project to familiarize myself with Sinatra so right off the bat I decided I needed to avoid any javascript (outside of bootstrap, styling everything is very time consuming). This also ended up creating a ton of restrictions that aren't immediately noticeable. Because of this I decided to tone down the scale that I was going to work on and went with a simple Videogame Library Tracker. 

The objective of this app is to allow multiple users to create, edit and delete games that they created and add any games that were created to their library. This way a user can use a game a different user created and manage the games they created.

## Structures
### Models
 * User
   * Has a firstname, lastname, email, password.
   * has_secure_password using bcrypt
   * Has many user_games, a join table for users and games.
   * Has many games, through user_games. Used to track their library.
   * Has many created_games, through foreign_key on Game object.
   * .approved?
     * Not used but can be used in the future if I wish to lock out certain Users.
   * .cleaned
     * Returns a cleaned OpenStruct object with the basic User Attributes in case I wish to store them a user in session.
   * #null_user
     * A user with empty attributes for logged out guests. Also prevents nil related errors.
 * UserGame
   * Has a User.
   * Has a Game.
   * Only used to join a User with a Game for their library
 * Game
   * Has a name, genre, publisher and description.
   * Has many user_games, a join table for users and games.
   * Has many users through user_games.
   * Belongs to creator which is an alias for the User that created it.

### Controllers
 * MyApp < Sinatra::Base
   * Application controller which all others inherit authentication and settings from.
   * Configures views and public assets directory to be within app.
   * Enables session_secret and sessions.
   * .authenticate
     * Takes in a session, finds the user it belongs to then authenticates the session.
 * SessionsController < MyApp
   * get 'login'
     * Renders the login page.
   * post '/login'
     * Accepts a email and password and logs a User in by setting their session.
   * get '/logout'
     * Finds user through their session, authenticates them then logs them out by clearing session.
   * get '/register'
     * Renders the register form.
   * post '/register'
     * Creates a new user and sets their session with the new users info.
 * ApplicationController < MyApp
   * get '/'
     * Renders a welcome Page.
   * get '/games'
     * Renders a page with all games in the app.
     * Allows a user to create, edit, delete games and add games to their library.
   * get '/game/:id'
     * Renders information about a Game.
   * get '/game/:id/edit'
     * Renders a edit page.
     * Verifies if the user is an admin, moderator or creator before rendering.
   * post '/game/:id/edit'
     * Accepts updated game parameters.
     * Verifies if the user is an admin, moderator or creator for processing.
   * get '/game/:id/delete'
     * Deletes a game. 
     * Verifies if the user is an admin, moderator or creator for processing.
   * get '/games/new'
     * Renders a new game form.
   * post '/games/new'
     * Creates a new game.
   * get '/library'
     * Renders a User's library
   * get '/library/:id/add' do
     * Adds a game to the User's library
   * get '/library/:id/remove'
     * Removes a game from the User's library
   * get '/failure'
     * Renders a failure pages. 
     * Accepts a @error hash.
       * {'error_type' => ['error_1', 'error_2']}

# Problems I ran into and their workarounds
### Href not being able to Post
When a User adds or removes a game to their library I felt that a Post request was much more appropriate then a get request because it was updating information. 

At first I tried to play around with creating a form with a submit button that is a text link. However the center of text in a text line is different from the center of text within a button. So even though I could remove the borders, change colors and add hover effects on the button I could not get it to center properly. I may have been able to play around with the css more to achieve my desired results. However I did not want to have any wierd unforseen styling issues pop up in the future, so I opted to just use a href and make it a get request instead. This was something that could have easily been fixed with an Ajax but in a effort to stay completely on server sided rendering I opted not to do that.

### Multiple Controllers
At first all of my Controllers inherited from Sinatra Base. I quickly learned however this is not DRY at all because I need to configure all of my sinatra/rack settings in each Controller and add my authentication method to each also.

I always feel iffy inheriting from a Class that I do not fully understand the inner workings of. There could be strange meta programming going on in the background that I am not aware of. Even if it is my class that is inheriting from another there are still a lot of unknowns about the class it inherits from.

However in this instance I ran into no problems having my Controllers inherit from a central MyApp Controller. If this were production I would have gone through the Sinatra source code or ask on a form. But since this was a personal project I opted to just try it out. Ended up working exactly the way I wanted it to be. My controllers all inherited the directory and session settings and authentication method from MyApp.

I later learned that this is the way controllers are set up in Rails. So I was pretty hyped I stumbled upon a Rails design decision on my own.

# Hosting
### https://videogame-tracker.lfp2.gg
I have the app currently hosted on my Google Kubernetes Engine cluster. It has 3 replicas and grabs the image from docker hub at cooljacob204/videogame-tracker.

I have a deployment running Jenkins that waits for a push from Github whenever a new commit is made on the master branch. When it receives that commit it builds a new docker image, pushes it to the hub then deploys it to the cluster. Thats basically it. The deployment controller has a readiness probe running at '/'. GKE does a rolling update of the 3 pods. If it doesn't detect the updated running app after the default amount of time Kubernetes will cancels the deployment and roll back any pods that were updated.
