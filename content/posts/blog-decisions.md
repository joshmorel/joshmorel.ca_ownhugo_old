+++
title = "Blog Decision Making"
date = "2018-01-27T23:11:00-05:00"
tags = ["Blogging", "Golang", "Hugo"]
slug = "blog-decisions"
summary = """
I write about my decision to switch to yet another (3rd so far) static site generator - Hugo.
"""
+++

## Background

I started this blog back in November 2016 with the aim of sharing some knowledge,
developing a web presence beyond social networks, creating some thoroughly written
technical references for my future self, and learning about different web technologies in the process.

While it's been full-steam ahead with learning about the web, my attention to this blog definitely been laggin.
My last post was half a year ago! I've narrowed down the why to a single reason.
Essentially, I've been questioning the chosen medium from a time-vs-value perspective for publishing content.
From a learning perspective, I'd learn most of what I could get out of the process having moved onto more advanced web projects. With regards to web precense and knowledge sharing, [Medium](https://medium.com) (with a capital M) made a lot more sense from practically every angle.
It would be much easier, with a larger audience, more opportunities for feedback, and at no cost (however I was only
spending 5 cents for S3 a month, which is practically free).

I didn't want to leave it languishing with years between posts, so it was time to make some decisions.

## Decision Making

My first decision was to use a new platform.

It was going to be Medium, [steemit](https://steemit.com/) or, if keeping it self-managed, a new static site generator.
There was nothing wrong with **Pelican** but I needed new energy and wasn't doing much in Python over the last year. Using a language I am engaged in would
provide an opportunity to contribute to the project, its themes, or plugins.

The next decision was keep it self-managed. My interest is not just in writing articles, but also hosting a personal portfolio and perhaps also a "knowledge base" of notes and code snippets around different technical topics.
I currently am doing this, but it's totally private in [Vimwiki]({{< ref "create-householdwiki-vimwiki.md" >}}) synced across a `Nextcloud` instance. If it makes sense to put on the web, why not?

I could not do all of this in Medium or steemit. Plus, I can always copy/paste some of the choice articles to reach a larger audience.

The final decision was to pick a static site generator.
The top two candidates were obvious for me, based on the languages/frameworks I was immersed in. [GatsbyJS](https://www.gatsbyjs.org/) as the most popular generator using React/JavaScript and [Hugo](https://gohugo.io/) as the most popular generator using Go. I say _was immersed in_ because for front-end frameworks I'm at least temporarily switching my focus to VueJS due to a project I'm working on. Mostly for this reason, I chose **Hugo**, although the fact that the workflow seemed a lot more similar to Pelican than GatsbyJS' helped nudge the decision.

## Hugo Impressions

I must say I'm pretty impressed. The [splash page](https://gohugo.io/) promotes it as the "world's fastest framework for building websites" and I believe it. Before Pelican I took a first, short-lived stab at blogging with Jekyll. I was impressed at the time with Pelican by its own performance boost compared to Jekyll, but Hugo is on a whole other level. If Jekyll is like a golf cart and Pelican a minivan, then hugo is a supersonic fighter jet. Live-reloading takes around 25ms where-as for Pelican, with the same amount of content, it's around 500ms. That's a _20x improvement_!

With regards to features. It seems, at worst, feature equivalent to Pelican. The themes, I'm confident, are better in Hugo. I want some more time to trial a greater variety but in just browsing over the day, I've found themes specialized for documentation as well as company and product pages. In Pelican, all the themes were made for blogging.

## What's Next?

I could write a how-to on migrating to Hugo from Pelican, but this has already been done many times (more typically from Jekyll it seems). 
I know this because many of the [theme demos](https://themes.gohugo.io/) include an article on just this topic.
Plus I've already done this kind of thing enough with Pelican and want to move onto writing about more advanced topics.

One to-do is exploring the themes and built-in functionality a bit more, perhaps with the goal to eventually create my own theme.

More importantly, I need to plan what I'll be writing about. I'm aiming to be more focused on a particular topic or technology this time.
Although I generally prefer to go broad rather than deep with my interests,
I think this would be a better way to distinguish the site. I'm pretty sure this is how [CSS Tricks](https://css-tricks.com/about/) started and now look at it!
One thing that's inspired me is this Medium article on [learning about blockchains by building one](https://hackernoon.com/learn-blockchains-by-building-one-117428612f46). There's definitely potential in building programs to emulate simplified aspects of different blockchains. Just reading about them is not quite enough to gain a good understanding, while trying to jump into a blockchain's source code would just be overwhelming at this point in my software development journey.
