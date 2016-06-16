---
layout: post
title: It's probably time for an update...
---

So more than half a year has elapsed since my last post, and I think it's about time that I blow away the dust that's collected atop this little blog and issue an update. Since December of last year, I've worked on a bunch of projects in my pursuit of web development, so I would like to use this post as an opportunity to quickly run through some of them.

Around this time last year, I began working my way through Free Code Camp's web development program. The program is broken into three sections, providing certificates to participants who complete challenges in front-end development, data visualization (with an emphasis on D3 and React), and back end development.

When I began the program I had been personally studying JavaScript for about a year and a half. I was also working full time at a small teacher's union as a technology specialist where almost all of my work involved managing data via queries and spreadsheets. At the time, my primary attention was directed at learning SQL so that I could extend the union's membership database in ways that would provide better analytical opportunities for union leadership. This was satisfying work, but it suppressed my interest in web development and slowed my progress through [Free Code Camp's curriculum](https://www.freecodecamp.com/thomasij813).

When I moved to Seattle in October, however, I was able to focus more directly on JavaScript development. I powered through the coding challenges required by Free Code Camp's front end program (they can all be viewed on my [codepen](http://codepen.io/thomasij813/)) and received a certificate of completion in early December.

In January, I began working through the back-end challenges, which required learning technologies that powered server side programming and REST APIs. First, I read and coded along to a few tutorials that covered Node.js, a topic I was somewhat familiar with from my experiences with NPM. I continued by learning about Express, an application framework for Node that greatly simplifies the process of writing back end code. Finally, I learned about MongoDB (and Mongoose), a popular noSQL database that stores data in JSON-like documents.

After receiving instruction in these areas, I felt well prepared to take on the first five challenges in Free Code Camp's back end program. These challenges involved writing small API microservices that respond to GET and POST requests. For example, one project required writing a URL shortening application, while another required developing an 'Image search abstraction layer' that  delivers JSON data containing image search results.

I've always been fascinated with REST API's, and I really enjoyed writing the code that powered these microservice applications. You can check out all five of them here:

* [Timestamp Microservice](https://dry-dusk-26167.herokuapp.com/)
* [Request Header Parser Microservice](https://lit-ocean-52245.herokuapp.com/)
* [URL Shortener Microservice](https://still-inlet-20198.herokuapp.com/)
* [Image Search Abstraction Layer Microservice](https://murmuring-atoll-28618.herokuapp.com/)
* [File Metadata Microservice](https://murmuring-hamlet-56604.herokuapp.com/)

After completing the microservice challenges, I decided to take a break from Free Code Camp and work on a [personal project](http://mcelroymadness.heroku.com/). I'm a big fan of podcasts, and in particular, I really enjoy listening to some of the podcasts affiliated with the McElroy brothers. The problem, however, is that the McElroys produce something like 20 different podcasts and youtube series, and there's currently no way to keep up with the latest episode of each of these productions without filling up your podcast app with subscriptions.

So In February I started working on an app that fetches the latest mp3 URL for each podcast (or video in the case of the youtube series) and loads it into the browser ready to play. The project involved setting up the front end (a simple list of podcasts with some light javascript to control mp3 playback) as well as the backend (an express server with an API endpoint that returns the sorted JSON data). It was a fun and challenging project, and I hope to expand it the future.

Rather than continuing with the backend challenges, in March I shifted my focus to tackling the data visualization track. The track involved completing five projects using React and another five using D3. I spent a lot of time in March reading through the documentation on React (and Webpack) and following along to some popular online tutorials. React is a great library, and its functional  and component-based approach to front-end engineering is a pleasure to work with. I'm in full agreement that new engineers should learn React simply because it forces you to think critically about how data flows through the components of your application.

I finished up the React projects by the end of March and began teaching myself D3, a popular data visualization library. D3 binds data to DOM elements (ideally in the form of SVGs but not exclusively) by making heavy use of method chaining. This approach adopts a somewhat declarative outlook that can be tricky for new developers to wrap their head around (at least it was for me!). In any event, it took me a few months to complete the D3 challenges and finish up the data visualization track (I received the certificate of completion on May 30).

Since then I've been working on some small projects here and there. I revisited the [pomodoro timer](http://thomas-johnson.net/pomodoro/) and the [weather application](http://thomas-johnson.net/weather/) I initially built as part of the front end track and refactored the code and design to make use of React and Sass. I have also been reading up on Redux, a state-management library created by Dan Abramov that heavily emphasizes functional programming and immutability and is popular among React developers. I'm actually very much inspired by the principles that inform Redux as well as the engineering techniques that it embodies, and I think that taking the time to really explore the project is helping me become a better developer.

I'm also currently working on a small package (less than 10kb) that helps create really simple HTML sliders. The project is nearing it's completion (I just need to write up the readme and refactor a little more), and I'll probably post a little more about it when it's finished.
