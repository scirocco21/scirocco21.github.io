---
layout: post
title:      "Building a social cryptocurrency portfolio app in Rails "
date:       2018-08-20 16:08:00 +0000
permalink:  building_a_social_cryptocurrency_portfolio_app_in_rails
---


For my Rails project, I built a simple portfolio app in Rails, which lets users create multiple portfolios (e.g. 'High-Risk', 'Low-Risk' etc.). Each portfolio in turn can have many positions, which is an abstraction for the amount of a coin and its value. This set up allows one to say that the value of a particular portfolio is the total value of positions it contains, for example. 

Coins (e.g. Bitcoin, Ethereum, etc.), on the other hand, are not accessible for the user, but can only be created by an admin. There's a great Ruby gem called rails_admin that quickly builds an admin dashboard. To make creating new coins as painless as possible, I used Nokogiri to scrape data like the ticker symbol and image of each coin from coinmarketcap.com. The admin workflow consists of no more than entering the name of the coin into the admin dashboard, which then becomes slugified automatically (to deal with tricky cases like "Bitcoin Cash', where the space creates trouble when building the URL). When the create method is triggered and a coin with the relevant name created, it is immediately assigned a symbol and ticker. Finally, each coin has a value attribute, which scrapes the dollar value from coinmarket.com every time a user looks at their portfolio. Note: this is a bit of a slow mechanism and wouldn't scale in a real application, and I would definitely want to use APIs here for the next iteration of the project. 

The app - called 'Coinopoly'- has a social element. I worked my through [ch. 14 of Michael Hartl's Rails Tutorial](https://www.railstutorial.org/book/following_users), where he sets out very nicely how to build a Relationship class that enables users to follow other users, and be followed by them in turn. To make the association of one user with another possible, one also needs to join the user table onto itself, effectively creating subset of users (e.g. the set of all those users whose followed_id is equal to the id of some user in the database).

With the social element set up, the next challenge was to do something with it! Here, I used another gem to help me out, called public_activity. The idea was to track the activity of users across the app, and then to build a follower_feed that would track the followees of a given user. 

**models/activities_controller.rb

```
class ActivitiesController < ApplicationController
  include UsersHelper
	
  def follower_feed
      @activities = PublicActivity::Activity.order("created_at desc").where(owner_id: current_user.follower_ids, owner_type: "User")
    render 'users/follower_feed'
  end
	
end
```

**views/users/follower_feed.html.erb

```
<% if @activites.nil? %>
  <h4>Not much has happened yet. Click <%= link_to 'here', users_path %> to follow more people.<h4>
<% else %>
  <%= render partial: 'public_activity/activities/activity', collection: @activities %>
<% end %>
```

**views/public_activity/activities/_activity.html.erb
```
<% if displayable?(activity) %>
  [<%= activity.created_at.localtime.strftime('%H:%M:%S %-d %B %Y')
  %>] <%= link_to activity.owner.name, activity.owner if activity.owner %>
  <%= render_activity activity %>
  <hr>
<% end %>
```

Setting up public_activity was quite tricky, as it would generate blank (and therefore useless) entries for activities that had been created and subsequently destroyed (e.g. Bob following Smith, and then unfollowing Smith later). I fixed this bug with a simple conditional statement in my relationship/_create partial:

```
<% if activity.trackable %>
  has started following <%= link_to activity.trackable.followed.name, user_path(activity.trackable.followed) %>
 <% else %>
  has stopped following a user
<% end %>

```

For anyone interested in the public_activity gem, I can recommend this episode of RailsCasts: http://railscasts.com/episodes/406-public-activity?autoplay=true

