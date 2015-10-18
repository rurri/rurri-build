---
title: "Shutting down my last server"
posted: 2015-04-26T14:10
tags: []
track: Technical
---
I have run a personal server for as long as I can remember. It started with 10BASE2 connections across my machines that I put together in middle school with the help of a neighbor,
and has been going strong ever since. It evolved into hosting sites over a cable modem (one of which maxed out the bandwidth for a while), then
to a machine running on a static IP in a dorm room at college. Things got serious after that, with co-location fees starting to come in for
my personal server. Finally, the co-location evolved away and I had a virtual servers--switching several times between AWS and RackSpace.

Now, I am shutting down my last server, hopefully to never start one again.

This is how it happened.

##Lost items
I don't really have a solid grasp of how many things I have lost over this time period. I know it is a large amount.
Somewhere in a closet is is a pile of floppy 5.25 inch floppy disks with backups from a wwiv bulletin board that I ran for a while.
It was popular and had a bunch of user information and messages on it. I doubt the disks work--assuming I could find them.

I wrote up and diagrammed climbing routes for Joshua Tree National Park. They didn't look too hot, but they were the only ones online for a while.
These are completely lost. These were hosted on Earthlink on an account that hasn't been paid for in about 20 years. Wayback machine has no record of them.

I have none of my projects from college. Of course this code wasn't the most useful, but I did write it, and it would be kind of fun to take a look at some of that code again.

I don't even want to think about the photos I lost. I may have had a backup on shutterfly once on an account I no longer remember.

Just a few examples, but the real shame in all of this, is that this was a *really* small amounts of data. And by small, I mean Megabytes type of small. It has to be,
because the hard drive was 20MB, so couldn't have been much more than that.

I can pinpoint the reason's that I have lost data, and it usually comes down to one of these:

* I kept only local copies of the information, and eventually lost them.
* I kept data on a server I operated and it was not moved to a new one during a migration
* I kept the data on a centralized service with a subscription and eventually forgot about it
* I kept the data on a centralized service and they are now defunct

Keeping a server running means I also need to worry about where that data is and how it is being backed up.

##Maintentance
It used to be interesting to get into the nitty gritty of Novell Netware when I was in 7th grade, but keeping track of software and OS
updates for a personal server and spending time applying them and testing them is just not a great way to spend an evening.

I didn't do this for a while, and some spammers were able to use one of my drupal sites to send out spam for a day before Rackspace caught it and shut it down.

Maybe there are some who like to install OS updates as a hobby, but I would rather be writing code.

##What changed
I need a place to serve my code, and so far this has always been server side code that has needed a place to run. Over time I have had personal servers running
Tcl, Java, PHP, Node.js. I need my personal websites; not because what I publish is an any kind of demand, but because I need a sandbox, a proving ground, a place for
experimentation. Writing something on my local machine for nobody to see is very anticlimactic, even if the alternative is publishing something that nobody sees.

A few developments that allow this change
* Container services for deploying server side code (Heroku)
* Client side applications (Polymer, Angular, React)
* SaS and Infrastructure as a service providers for almost anything you could need

##Solutions
Clients do more work. The code that used to run on the server can run on the client--at least for personal projects. This means, that the remaining
problem is for the client to be able to fetch and store the data it needs. This can be tricky to do and not run a server, but there are so many
service providers out there that using these specialized services to store data and then fetch it via an API on the client side is now a real
possibility.

A few possible data storage/retrieval problems and their possible solutions are:

Blog Posts:
 * Markdown and static generators
 * Use blogger or other service and fetch posts via an api

Comments:
 * Disqus
 * Facebook Comments

Videos:
 * Embed via Vimeo
 * Embed via Youtube

Photo Sharing
 * Flikr API
 * Picasa API

Evolving Lists
 * Listly
 * statically served JSON

Keeping the site off the server and on the client, does mean forcing ourselves to think about what services already exist and how we can use them.
Remember when everything was going to be exposed via a SOAP endpoint and that this was going to revolutionize the web? Well it happened slowly, and
thankfully not using SOAP. With enough thought we can find a service and an API to serve the data we need in the form that we need it, and to let the client
do the heavy lifting.

##Not for everything
Of course, I would never dream of writing sites this way for my employers. We need the fine grained control, we need good SEO, and we need the site
to be accessible to a wider audience. I don't think it will be too long before the Single Page Application and client side applications are everywhere,
but it won't happen until they are faster and more accessible. In the meantime, they are a pleasure to work with for my own site, and this means, a simple
sync to an S3 site or a github pages account instead of the weekly server maintenance.

