---
layout: post
title:      "Subreddit Scraper Project"
date:       2019-01-19 22:38:07 +0000
permalink:  subreddit_scraper_project
---


Welcome to the first post of my blog. Today I will be writing about my Subreddit Scraper Project, going over it's general structure.

My project is split into three main parts:
* My Models
* My Scraper
* My CLI Menus



## Models
I have 4 Models that represent the information on a reddit page.

##### Subreddit
* Has a name, has_many Posts, URL and description

##### Post
* Has a title, belongs_to Subreddit, belongs_to User,  created date and link

##### User
* Has a name, has_many Posts

##### Comment
* Has a comment, User and Post

All of these models also have  a class variable @@all which keeps track of all of their instances.


## Scraper
My scraper actually did start off originally as a scraper. I had it crawling through Reddit and scraping the HTML off of the pages. However it quickly became rate limited and to avoid being banned I did not try to make it look more like a web browser doing the requests.

So I began to look at alternatives and read about [Reddit's API](https://www.reddit.com/dev/api/). Awesome, until I started to look further into it. It has a whole lot of rules about what you can and can't do, requests per minute limit and other weird restrictions.

However it also lists the wrapper [Redd](https://github.com/avinashbot/redd) on its wrapping page. It handles requests limits and other stuff for me. Awesome, now I am getting somewhere.

So now my Scraper class consists of 1 class method called scrapeSubredditFromName(name) and is only 39 lines long. Not bad for pulling all posts, users and comments from a Subreddits front page.

## CLI Menu

My CLI menu consists of 2 primary parts.

### Cli Class
The CLI class is about as short as a class can get. It has one function which clears the console and displays a welcome message.

It inherits most of its functionality from my Menu Module.

### Menu module
This is probably the most interesting part of the way I structured my project. Originally this was one giant class within the CLI class. I decided for readability and to get as close to the [SOLID Principle](https://en.wikipedia.org/wiki/SOLID) as reasonable on a project like this that it was best to split it out into different files, each one being one of the menus.

I then however ran into an annoying issue.

In my CLI class I had two chunks of code. At the top I had: 


```
def self.before(*names) # used to append methods
  names.each do |name|
    m = instance_method(name)
    define_method(name) do |*args, &block|  
      yield
      m.bind(self).(*args, &block)
    end
  end
end
```

And at the very bottom I had was:
```
before(*instance_methods){Gem.win_platform? ? (system "cls") : (system "clear")}
```

self.before(\*names) allowed me to append methods with code and insert it at the top of methods. Cool little code snip-it I found on Stack Overflow.

And at the very bottom of my class I would run ```before(*instance_methods){block of code}``` and feed it a code block of whatever functionality I wanted to add. In this case it would be to clear the console every time a new menu function was called.

When I pulled my my menu out into different modules I realized I would have to insert this code into every file. Bleh, I want to repeat myself as little as possible.

So at the time I had 4 modules:
* MainMenu
* PostMenu
* SubredditMenu
* UserMenu

Some also had shared functions which had to be repeated or thrown into a seperate module.

So I decided to create a Menu class which would contain the shared classes and make it inherit all functionality from the other classes.

```
    include MainMenu
    include SubredditMenu
    include UserMenu
    include PostMenu
```

Then after I included those classes I can run my ```before(*instance_methods){block of code}```  method and it would insert my console clearing code into all additional functions.

One downside I found is that because I put shared functions into the Menu module the other menu Modules cannot work without the Menu Module. However I cannot include the Menu module in each class because I wouldn't be able to apply my clear console code chunk to them in one easy line.

It's a slight downside but because everything is intended to be bundle together I think it is an acceptable tradeoff.


