---
layout: post
title:      "Final Rails Project - This was a tough one"
date:       2019-04-15 14:29:56 -0400
permalink:  final_rails_project_-_this_was_a_tough_one
---


Today I completed my final Rails project, or at least got it working decently enough to turn in...

This was definitely way more of a struggle than the final Sinatra project. In the weeks leading up to project week, I was feeling relatively comfortable with the material in the labs, although it was definitely a bit harder to grasp than some of the previous material covered. But once I attempted to actually build a Rails project from scratch, things became way more complicated.

Things started off pretty well. I wrote an outline of what I would need to make the site function how I wanted it to, built the necessary models and relationships, and added the proper tables to the database. My first major hiccup came when trying to add OmniAuth to allow the user to login with a Facebook account. At this point I had already built a more standard login fucntion using Bcrypt and `has_secure_password`. When I tried to add OmniAuth in addtion to this, things started breaking left and right. After hours spent Googling, I was able to figure out a solution that allowed for both types of login, but it seems pretty ugly and I am convinced there is a much better way to do it. 

This turned out to be a common theme throughout the rest of the project. One other project requirement was added a nested form that allows for the creation of an object on the 'new' page of a different object. This also caused several issues for me, especially when I was attempting to add custom logic to the creation process rather than using `accepts_nested_attributes_for`. Again, I was able to come up with a solution that seems to work, but looks very ugly. 

Several other seemingly random errors that required intense Googling to solve would keep popping up every time I tried to add a new feature to the site, which made it difficult to work but also provided a great learning experience. 

While at certain times it was pretty frusturating to work on this project, now that it is "completed" I'm happy that I went through the process. I have always learned best by actually doing, rather than reading or watching a tutorial, and working on this project taught me a ton. I am excited to review the project with the lead instructor for my cohort and see how I could have done things better. Im sure it will be another eye-opening and very helpful experience

