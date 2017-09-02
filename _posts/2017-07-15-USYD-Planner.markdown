---
layout: post
title:  "A course planner for USYD"
date:   2017-07-15 13:48:34 +1000
categories: assessment
---


A project I've been working on that attempts to make mapping out which subjects to take more straight forward.

Live page is [here](https://usyd-planner.herokuapp.com/).

Source code is [here](https://github.com/jwhardwick/Course-Planner-Vuejs).

It's a single page web app using a Node/Express/Vue stack that connects to an SQLite database. It's been a great learning experience building this, and it's still an ongoing process. Throughout the build I've shifted from Postgres > MongoDB > DynamoDB on AWS, and now I'm keeping it simple and using SQLite, which provides by far the fastest experience.

My favourite feature to implement was the autocomplete, which I coded from scratch and still amazes me everytime I use it. It's definitely needs optimisation work as it doesn't take advantage of caching to reduce database queries, but for the time being it does its job.

It's also helped me improve my Git abilities, as I've tried to keep my commits modular (for the most part...) and work on new features in a seperate branch. 
