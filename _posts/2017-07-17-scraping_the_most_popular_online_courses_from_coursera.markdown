---
layout: post
title:  "Scraping the most popular online courses from Coursera"
date:   2017-07-17 13:50:56 -0400
---


This week I finished my first project on Learn, a CLI app that scrapes data about the most popular open courses from Coursera. The app returns 20 of the hottest offerings at the moment with some basic details (Which institution offers the course? What's the link?) and lets users request a detailed description. 

It's pretty interesting what one finds among the top 20: top of the list is Andrew Ng's Machine Learning course at Stanford, followed by UCSD's ever popular 'Learning how to learn' . A surprising runner up is Princeton's 'Bitcoin and Cryptocurrency Technologies' , which (at fourth place) goes to show how far crypto has entered the mainstream by now. 

Building the app itself was a lot of fun: thankfully, Coursera has arranged their data in a way that is quite accessible, with each course occupying its own div. container. More challenging was working out what I wanted the app to do and putting together a simple but functional CLI call method; in the end I settled for two levels of depth. With only three main classes, a scraper, a CLI class and, of course, the course class itself, assigining responsibilites for each task was manageable and only caused the occasional hickups. 

Eventually it would be fun to extend the app to include data from more than one website, so as to include some of the other big sites like EdX or FutureLearn. But this would throw up the issue of ranking popularity not just on one site but across different sites. I've left that can of worms unopenend for now! 

Another element of functionality that is missing from the app (and Coursera's list) is the option to arrange popular courses by subject. For the user interface it would be more intuitive to enter first the general area of study one is interested in, say Science or Technology, to then be given the courses assigned under that rubric. 

I've kept numbers down to twenty, but if anyone wants to see a big bad list of 50 (which I'm not using in the app), check this out:

http://www.onlinecoursereport.com/the-50-most-popular-moocs-of-all-time

So little time, so much to learn!

https://github.com/scirocco21/popular_moocs


