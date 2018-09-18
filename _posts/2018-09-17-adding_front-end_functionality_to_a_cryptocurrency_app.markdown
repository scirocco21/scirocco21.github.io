---
layout: post
title:      "Adding front-end functionality to a cryptocurrency app"
date:       2018-09-17 22:50:54 -0400
permalink:  adding_front-end_functionality_to_a_cryptocurrency_app
---


Adding a JS/Jquery frontend to my Rails Rails project (a portfolio tracker for crypto assets:  more details here: https://scirocco21.github.io/building_a_social_cryptocurrency_portfolio_app_in_rails) has proved to be challenging; below I go into details about some of the more interesting problems I encountered along the way.

## Building templates with Handlebars

Templates are great, and Handlebars.js offers a simple way of injecting the properties of model objects into a template and then rendering the resulting HTML to the DOM. One of the drawbacks of Handlbars, however, is one of organization. It's no fun to have lots of <script> tags cluttering the view and makes organizing one's code cumbersome and diffcult. 

The best solution to this I have found is a little gem called '**handlebars_assets**', which integrates Handlebars with Rails' asset pipeline. Once the gem is bundled, templates can be organized into separate folders in 'assets/javascripts/templates. Creating the templates also becomes very straightforward. A collection of books, for example, can be used as the context for a template named book_list with only one line of code: 
`HandlebarsTemplates['book_list']({books: books})`

## Setting up 'previous'/'next buttons

Suppose a user has many portfolios, and that the portfolio show page includes all the positions (coins and their quantity) that make up the portfolio, as well as a form to add a new position. Now if we want a user to be able to cycle through all their existing portfolios by way of 'previous/next' buttons, quite a lot needs to happen!

Take just the 'next' button as an example. Once the button on the first portfolio's show page, say, is clicked, our nextButton() function needs to request the next data portfolio (portfolio 2), create templates from that data, and, once the 'get' request to the server is completed, reset the buttons on the updated show page, now with portfolio 2's data showing. Since a user with only two portfolios should not be able to see a 'previous' button on their first portfolio show page or a 'more' button on their second portfolio show page, we need another function to handle which buttons are showing for each click of the button ( `setButtons()` ) :

```
function nextButton () {
    $(".js-next").on("click", function(e) {
      e.preventDefault()
      let nextId = parseInt($("#next-button").attr("data-id")) + 1;

      $.getJSON("/users/<%= @user.id %>/portfolios/" + nextId, function(data) {
        updateDataAttributes(data["id"])
        let portfolio = new Portfolio(data);
        portfolio.injectNameValue(this)
        portfolio.renderPositions(findTemplate());
      }).then(setButtons)
    })
  }
```
```
function setButtons() {
    $.getJSON("/users/<%= @user.id %>/portfolios/", function(data) {
      let ids = data.map(portfolio => portfolio.id)
      let nextID = parseInt($("#next-button").attr("data-id")) + 1;
      let backID = parseInt($("#back-button").attr("data-id")) - 1;
      if (ids.includes(nextID)) {
        $("#next-button").html("<a href='#' class='js-next float-right btn btn-secondary'>Next portfolio</a>")
      } else {
          $("#next-button").html("")
      }
      if (ids.includes(backID)) {
        $("#back-button").html("<a href='#' class='js-back float-left btn btn-secondary'>Previous Portfolio</a>")
      } else {
        $("#back-button").html("")
      }
    }).then(nextButton).then(backButton)
  }
```

The trickiest part is to get the sequence of events right here: the function setButtons() can only fire once nextButton() has been completed, but equally, nextButton() should not fire until setButtons() has successfully decided which buttons should be displayed on the page. What is doing the magic here are the` then()` snippets of code, which organize the asynchronous getJSON requests in an execution order that makes sense for the app. 

## Task scheduling with Rufus
A must have for any cryptocurrency portfolio app is an accurate representation of the currrent price of the assets. How can one make sure that the price stored in the database for Bitcoin, for example, falls within an acceptable range of accuracy, say, within 10 minutes of the market price? 

One way to solve this problem is to use, once more, a Ruby gem: the Rufus (why??) scheduler. For my Coin model, I would set up something like the following:

```
  require 'rufus-scheduler'
  scheduler = Rufus::Scheduler.new
  scheduler.every '600s' do
    Coin.all.each do |coin|
      coin.set_value
    end
  end
```

The `set_value ` method itself makes an API call to a well-know crypto site via Rails' NET::http and assigns the result to a given coin's `:value` attribute:

```
  def set_value
    url = "https://min-api.cryptocompare.com/data/price?fsym=#{self.ticker}&tsyms=USD"
    uri = URI(url)
    response = Net::HTTP.get(uri)
    price = JSON.parse(response)['USD']
    self.update({value: price})
  end
```

Alas, all this updating presents quite a burden to the server and may not be a scalable solution for a larger app. Still, it's one step closer to the goal! 





