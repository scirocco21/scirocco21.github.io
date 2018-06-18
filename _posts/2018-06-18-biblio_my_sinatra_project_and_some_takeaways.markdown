---
layout: post
title:      "Biblio: my Sinatra project and some takeaways"
date:       2018-06-18 04:07:54 +0000
permalink:  biblio_my_sinatra_project_and_some_takeaways
---


After much rumination over the best domain for the project, I decided to work on something that I'd use myself: a simple app that functions as a digital bookshelf, allowing users to create book objects and to annotate them. 

Wiring up the correct associations was a bit of a challenge initially, as I assumed that books and users could simply be related through booknotes, such that a users and books have many booknotes, and users have many books through booknotes, and books have many users through booknotes. This simple idea did not end up working at all, as I also wanted users to be able to create books without any annotation. 

What did the trick in the end was drawing the whole domain on a whiteboard, and later on gliffy. This is what worked for me:

![](http://)https://ibb.co/mswxhy

When it came to drawing the appropriate routes and developing the views, the trickiest part was thinking about duplicate data and user permissions. If user A, for example, creates 'Don Quixote', and this book is publicly viewable, user B can then add his or her own booknote on 'Don Quixote' without having to create the book object again and there being two books with the same title and author in the database. 

But what happens if user A decides to delete the book that he or she created? Will user B's booknote disappear with it? My solution was to make each book and each booknote specific to the user, and to give each user permissions to edit and delete. 
