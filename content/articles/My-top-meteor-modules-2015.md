---
title: "My top meteor modules (2015)"
posted: 2015-04-30T05:13
tags: [meteor]
track: Technical
---

A big fan of meteor right now. Making apps in meteor can be quite a pleasure. Here are the modules I almost always end up using.

## stevezhu:lodash ##

The underscore library is pretty universally used, especially by the meteor community. This is most likely a result of many of the initial tutorials using underscore in their examples and getting users going with it. I love it as well, so why not use this drop in replacement for underscore which does everything underscore does but slightly faster.

>_ and lodash are both defined as lodash if used in a package. In an app, _ is redefined as underscore so you may need to have _ = lodash; in some main javascript file.


## iron:router ##

Firs thing I install, and judging by most examples, tutorials, and walkthroughs I am not the only one. Iron router is a pretty integral part of the way I use meteor, and I think all but the most esoteric applications will benefit from using it.


## meteorhacks:npm ##

There has been not just a little discussion on meteor’s decision to use its own package manager instead of just using the already popular npm. I won’t go into too much detail, but I *can* see the benefit of having modules specific for meteor in that it allows the module to do many things in a meteor way, and more deeply integrate into the application.

The big miss though is that there is *no* support for npm out of the box. Forcing people to wrap npm modules in a meteor module is a big mistake in my opinion. Someone has to manage this repacking, which is a big waste of time, and now as a consumer of this repackaged npm I am at the maintainers mercy of how often they repackage the npm module.

Meteor should allow for meteor packages **and** npm modules.

This module takes care of that, allowing you to have a package.json file in your root directly where you can easily add npm dependencies and use them. This npm packages are downloaded during build time and appropriate .gitignore files are setup so that the dependencies don’t get checked into revision control.

The package also plays nicely with demeteorizer and all dependencies get exported nicely as dependencies in the demeteorized app.


## meteorhacks:async ##

Companion module to meteor hacks npm module. Once npm packages start being used there are usually a lot more async calls happening in the app. Meteor hack’s async module helps manage these.


## nemo64:bootstrap less ##

I am a big fan of using less, so when I use bootstrap for a project, I love the way this module sets up bootstrap for me as less compilables. I can turn on and off bootstrap components via a json config file, and do fun things with the bootstrap theme such as tweaking less variables and theme colors without saving hardcoded css files.

Setup is pretty simple, install the module and then create an empty custom.bootstrap.json file somewhere in your project. (/client/lib/custom.bootstrap.json for example)


## aldeed:collection2 ##

So now that you are finally free from schemas its time to install a package for enforce a schema. No really. aldeed's collection2 and simple-schema are great. You can enforce just the small bit of schema that you want on your collections and still be free to have wide open schema-less areas on the same collection.

I find that I start completely schema-less and slowly add in schemas as they become integral to how the application runs. Keeping my data nice and consistent while still giving me flexibility.


## aldeed:autoform ##

This is the missing form generator, validatorator and form event library. Create forms and their validation rules declaratively in configuration and watch the forms both be generated and automatically validated.

Once a schema starts taking shape from aldeed's collection2, you can create all sorts of admin pages and beautiful CRUD pages based on the schema. These can save a tremendous amount of work, and makes meteor’s rapid development even *more* rapid.


## yogiben:admin ##

Also dependent on some sort of schema being defined by collections, but very powerful. If your schema starts to take shape, you can install yogiben’s admin module and have beautiful CRUD pages for all your collections.

The module is pretty powerful and also looks great.

There are a lot of hooks and configuration options and ways to add to and tweak your own admin pages, but I found these either difficult to understand, not documented well enough, or not enough working examples to get a ton of customization in the admin pages without getting frustrated.

Out of the box, however the admin module supplies a great admin UI, so you might not need to customize it all that much.


## xolvio:cucumber ##

This is a no brainer for me since I like the format of cucumber tests. So bringing cucumber in as part of the velocity framework didn’t require much convincing.


## aldeed:tabular ##

If you ever need to display all or part of your collection in a table I highly recommend checking out this module.

The module just needs to know what columns of a collection you want displayed, what subset of the collection to show, and who can see it. Once this is done, you can include the table in any template and it is automatically setup with: publications, subscriptions, ACL, search, sorting by columns, and pagination.

A great starting place for more custom admin interfaces.
