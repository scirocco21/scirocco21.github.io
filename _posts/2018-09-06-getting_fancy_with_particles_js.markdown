---
layout: post
title:      "Getting fancy with Particles.js"
date:       2018-09-06 23:38:04 -0400
permalink:  getting_fancy_with_particles_js
---


A great tool for creating a visually appealing landing page is particles.js, a lightweight library that sends any number of little balls (or stars, or squares, or whatever icons take your fancy) up and down your screen. Some of the functions are really cool: particles can become 'attracted' to the cursor, or be repelled by it, for example. 
		![](https://ibb.co/hfBjHz)

Integrating particles.js into the asset pipeline is only slightly more involved than getting started with Bootstrap. However, while it is enough to include a link to the external Bootstrap files in a script tag of one's index.html, for example, this will not work with particles/js, and it (in Rails) it is easier to just download and add the relevant files to assets/javascripts. The relevant files are particles.min.js and particles.json. The json file is the one that ultimately carries all the configurations (how many particles, what shape, etc.):

```
{
  "particles":{
    "number":{
      "value":400
    },
    "color":{
      "value":"#fff"
    },
    "shape":{
      "type":"circle",
      "stroke":{
        "width":1,
        "color":"#ccc"
      },
			
			...
}
```

Here are the steps to get up and running once the two files are included in your application:
1. require 'particles.min' in your application.js manifest: `//= require particles.min`
2. include `<%= javascript_include_tag 'particles.min', 'application'%>` in index.html.erb
3. still in index.html.erb, include 
```
<script>particlesJS.load('particles-js', 'assets/particles.json', function() {
     console.log('callback - particles.js config loaded');
    });</script>
``` 
4. create a hook for your canvas, e.g. a div with an id of 'particles-js' in your index.html.erb

5. in config/initializers/assets/rb, add these two lines to precompile the javascript code: 
```
Rails.application.config.assets.precompile += %w(particles.min.js)
Rails.application.config.assets.precompile += %w(application.js)
```

Congrats! You're good to go. Looking fancy already, right?

Resources:

Create Particle Effects With Particles.js
https://www.youtube.com/watch?v=qK3cgD09Qf0&t=879s

Interactive tool:
https://vincentgarreau.com/particles.js/

Github:
https://github.com/VincentGarreau/particles.js/




