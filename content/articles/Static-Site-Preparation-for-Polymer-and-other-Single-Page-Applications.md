---
title: "Static Site Preparation for Polymer and other Single Page Applications"
posted: 2015-05-03T13:11
tags: [polymer, grunt, metalsmith]
track: Technical
---
I have been recently working quite a bit with single page applications, and I love putting the same technology to use for personal projects, that I use on a daily basis for professional projects.

This means I get to play around with Single Page Applications—namely Polymer. The challenge was finding a way to use it with a static site hosted on Amazon S3 as a static site, but that still has new content added every few days, just like one might do with a static site generator.

## How is site generation with Polymer different ##

Single page applications don't have separate pages per article, but will consume data using an API. Since our data is static, and changes only when new articles or other pieces are added, we can create a static api. Storing the the pertinent data as JSON files and then retrieving it via get calls. Because of these differences, our static site generator will be creating JSON files not html files. This means that unlike a static site with html, when using a site generator with a single page application means wrangling data and creating easily consumable JSON files instead of creating statically generated pages. The html files will just be the polymer application, and will change only when we make changes to the actual application.

This means that we can divide our application in to two main parts:

* The application itself (in this case written in polymer)
* The content and data being served to the user and displayed in the single page application

## Similarities with other static site generation ##

Articles will still be written in markdown, and We still will have files generated from these source markdown files. We will still need a place to host these files such as GitHub pages or an S3 bucket.

## Other static site generators.  ##

There are many existing static site generators out there. Picking the right tool can be hard, but here are a few that I eliminated because they focused on generating html files from templates, and what we are looking for is more of a set of tools that can wrangle our markdown files with yaml headers into sets of JSON data that we will want to retrieve. Here are a few of the options that I looked at and ruled out. If you are not using polymer or other single page application framework to build your site, than any one of these will probably be a better choice. For a more comprehensive list of static site generators ranked by usage, and filterable by language, try [this list of static site generators](https://www.staticgen.com)

* Jeykll
* Octopress
* Hexo

## Enter Grunt ##

Grunt is already in extensive use by many single page application developers. Grunt is versatile, and written in javascript, a language that you are already using if you are writing a single page application. The best part about using grunt is that it is a well used build tool for client side web code and not a specialized static site generating tool that might not have long term support. It has a lot of users, developers, and documentation and is a great choice for generating the files that we need.

If you have not used grunt before, it is quite simple to pick up, as it is nothing more than a bunch of javascript code that gets called based on the configuration file called Gruntfile.js.

![Polymer, Content, and Grunt output](content/images/PolymerGrunt.png)

## Metalsmith ##

Metalsmith is a more specialized static site generating tool, but it has a grunt plug-in that allows it to be called using grunt. This means we can use many of its specialized processing for generating static content or JSON files, but still easily tie it into our grunt build and deploy process.

## The JSON output from Grunt ##

Since our polymer app will be properly vulcanized by grunt and ready to run, what we really need to focus on is getting the content in a readily readable state that we can bring in by JSON. I split my data up into several JSON files:

* A JSON file per article, with all meta-data, including the content as html. The JSON file should be the name/id of the content, so that it can be requested whenever needed by the client application.
* A JSON file of the most recent article(s) for use in showing on the home page. This facilitates a fast home page load of the most recent n articles. This filename will always be the same. Something like: last5articles.JSON
* A JSON file containing a summary of the metadata for use in searching. I keep the fields I index small, so that this file doesn’t grow too large. This file might just contain the title, tags, and id of each article and can be called something like: allArticles.JSON.

As the application developer, we can see that if these files were to exist in a known location, writing our single page application to consume these is now quite trivial.

In Polymer, to load in one of these files we now just need to include this snippet for getting a specific article:

```
<core-ajax id=“articleRequest”
           auto
           url=“content/articles/{{articleId}}.JSON”
           handleAs=“JSON”
           response=“{{article}}”>
</core-ajax>
```

And this snippet for getting all of our article data:

```
<core-ajax id=“allArticlesRequest”
           auto
           url=“/content/articles.JSON”
           handleAs=“JSON”
           response=“{{allArticles}}”>
</core-ajax>
```
The import thing to notice here is that all of our content is retrieved from JSON files at runtime by the client. This means that our application is completely content agnostic. Adding new content means only updating these JSON files. These JSON files could theoretically be served by an API somewhere, but in this case since I don’t want to pay for a server to serve up files that only change when I make a new article, I auto generate them with gulp and serve them statically.

## Grunt configuration for JSON files ##
All that is left is to actually generate these JSON files and make them a part of our distribution.

As a starting point I use the yeoman Polymer scaffolding for grunt. I think this is a great starting point for getting going on Grunt with Polymer and starts you off with some great tools and scaffolding for both running your polymer apps locally (`grunt serve`) and building your app for distribution (`grunt build`).

Yeoman’s polymer scaffolding is available on [github](https://github.com/yeoman/generator-polymer).

To generate thes JSON files I use metalsmiths grunt plugin available as an [npm module](https://www.npmjs.com/package/grunt-metalsmith).

Once installed, adding in this config to the Gruntfile will get you started.

The key metalsmith module I use to generate these is called `metalsmith-writemetadata`, and all of the config options can be  viewed in their [README on github](https://github.com/Waxolunist/metalsmith-writemetadata)

```
    metalsmith: {
      articles: {
        options: {
          metadata: {
            hello: ‘world’
          },
          plugins: {
            “metalsmith-filepath”: {
              “absolute”: false,
              “permalinks”: false
            },

            ‘metalsmith-collections’ : {
              articles: {
                pattern: ‘articles/*.md’,
                sortBy: ‘posted’,
                reverse: true
              },
              last5articles: {
                pattern: ‘articles/*.md’,
                sortBy: ‘posted’,
                reverse: true,
                limit: 5
              }
            },
            ‘metalsmith-markdown’ : {
            },

            “metalsmith-excerpts”: {
            },
            ‘metalsmith-writemetadata’ : {
              pattern: [‘**/*.md’, ‘**/*.html’],
              bufferencoding: ‘utf8’,
              ignorekeys: [‘stats’, ‘mode’],

              collections: {
                articles: {
                  output: {
                    path: ‘articles.JSON’,
                    asObject: true,
                    metadata: {
                      “type”: “list”
                    }
                  },
                  ignore keys: [‘contents’, ‘next’, ‘previous’,
                    ‘collection’, ‘mode’, ‘stats’]
                },
                last5articles: {
                  output: {
                    path: ‘last5articles.JSON’,
                    asObject: true,
                    metadata: {
                      “type”: “list”
                    }
                  },
                  ignore keys: [‘collection’, ‘mode’, ‘stats’]
                }
              }

            }
          }
        },
        src: ‘content’,
        dest: ‘<%= yeoman.dist %>/content’
      }
    }
```

## Deploying the app ##
Once the build is complete, the distribution folder will contain all the files needed to get going.

Additional modules can be used to automatically diff and then upload to an S3 bucket for serving to the public.
