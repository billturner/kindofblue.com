---
layout: post
title:  "How I'll build the new Lists of Bests"
date: 2017-11-02 21:06:00 -0500
description: "Explaining what I think I'm going to use in building the new version of Lists of Bests"
tags:
- lists-of-bests
- postgresql
- projects
- react
- ruby-on-rails
---
When Lists of Bests launched in 2003 ([see previous post](http://kindofblue.com/2017/10/bringing-back-an-old-friend/ "Post explaining bringing the site back.")), the site consisted of a few Perl scripts and _maybe_ an external CSS file. I don't think I used a bit of JavaScript on the site at all, and my design skills were less than competant.

Despite all that, the site did quite well. I can't recall the total number of users I had on the site at the time I sold it to the Robot Co-op, but it was maybe a few thousand. And that was **many** more than I had ever expected. All in all, I considered the project a success.

The web development landscape has changed quite a bit since that time, and I have some new decisions to make, when it comes to how I'm going to put the site back together again. I suppose using Perl again is an option, but I've forgotten everything about that language.

I think for the second version (well, third, if you count Robot Co-op's version) of the site, I'm going to split the front-end and the back-end into separate pieces. That's how we build things at [work](https://cloudpassage.com/ "CloudPassage home page") and it seems to work well. So, given that, I needed to make a decision on which direction to go.

For the back-end, I was wondering if I should go with a JavaScript server like [Express](https://expressjs.com/ "The Express.js site"), or stick with what I know and use [Ruby on Rails](http://rubyonrails.org/ "Ruby on Rails site"), and specifically using the new-ish [Rails API-only](http://guides.rubyonrails.org/api_app.html "Explaining the Rails API ability") functionality.

On the front-end, I was a bit less sure. I've already built a small [React](https://reactjs.org/ "The React JS framework site") application, but at work I've primarily worked with [Angular](https://angularjs.org/ "The Angular JS site"). At this point, I'd say I'm _probably_ a bit more proficient with Angular, but I'd like to attempt a larger project with React. And then there's also [Vue.js](https://vuejs.org/ "The Vue.js framework site"), which looks interesting and has gained a lot of steam lately.

In the end, based on experience and ability to find online documentation and examples, I think I'm going to go with React for the front-end, and Rails for the back-end. I think I'll be able to spin something up quicker with these than with any other technology.

Finally, for the original I stored all the data in the [MySQL](https://www.mysql.com/ "The MySQL site") database, but I've been working with [PostgreSQL](https://www.postgresql.org/ "The PostgreSQL site") the last six or so years, and I'm a little more comfortable with it.
