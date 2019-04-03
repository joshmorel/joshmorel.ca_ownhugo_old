+++
title = "Create a GitHub Personal Page with Python"
date = "2016-11-25T07:41:00-05:00"
lastmod = "2017-02-26T09:44:00-05:00"
tags = ["Blogging", "Python", "Pelican", "GitHub"]
slug = "create-a-github-personal-page-with-python"
summary = """
The second static site generator I used was the Python package Pelican, which is a good alternate to Jekyll for creating a GitHub Personal Page. My post summarizes a few of the benefits as well as how to use it in a basic way.
"""
+++

## Background

[Pelican](http://docs.getpelican.com) is a Python package which helps
you generate static sites. [GitHub Pages](https://pages.github.com/)
will host that static site for you for free. These two combined allow
someone to create and present a beautiful site without needing to know
much HTML, CSS, or JavaScript or require a paid or local web server.

Additionally, you can avoid [vendor lock-in](https://en.wikipedia.org/wiki/Vendor_lock-in) associated with using something like Wordpress as all your content will be in simple marked up text files maintained both locally and on GitHub.

Prerequisites include:

1. [Python](https://www.python.org) (2.7 works for now)
2. [Git](https://git-scm.com/)
3. A [GitHub](https://github.com/) account
4. The Pelican package

I'll leave the first three to you.

Before continuing I first must acknowledge [this
walkthrough](https://fedoramagazine.org/make-github-pages-blog-with-pelican/)
from Fedora Magazine. Mine is similar but with a few additions and
instructions which should work on any OS.

## Pelican

You can install Pelican with pip or your package manager on some Linux
distribution.

On Ubuntu try:

```shell
sudo apt-get install python-pelican
```

With pip (in Windows, for example):

```shell
pip install pelican
```

Restart your shell so you can use pelican utility commands. There's a
lot more info on their [docs site](http://docs.getpelican.com) so if you
have challenges head there.

## GitHub Pages init

To create a GitHub pages site you need to at least create a new
repository named `you.github.io` (where `you` is substituted by your
GitHub user name). To support the Pelican project workflow we will
create two:

1. `you.github.io-src` will contain the Pelican project used to
   generate the static website
2. `you.github.io` will contain the website itself output from a
   Pelican project

[Create](https://github.com/new) these repositories on GitHub, being
sure to initialize with a README to facilitate cloning locally.

Clone the source repo into a ghpages directory.

```shell
git clone https://www.github.com/you/you.github.io-src ghpages; cd ghpages
```

Add the website repository as a [git
submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules) in the
project's output directory.

```shell
git submodule add https://www.github.com/you/you.github.io output
```

## Starting the Pelican Site Project

You can easily get started with the pelican quickstart utility:

```shell
cd /path/to/ghpages
pelican-quickstart
```

There will be several prompts, use similar answers to the following.
Note I am not using either Fabfile or Makefile for automation. These are
more useful if publishing to a personally managed server (i.e. not
GitHub).

* Where do you want to create your new web site? **[.]**
* Do you want to specify a URL prefix? e.g., `http:example.com` **Y**
* What is your URL prefix? (see above example; no trailing slash) **http://you.github.io**
* Do you want to generate a Fabfile/Makefile to automate generation and publishing? **N**
* Do you want an auto-reload & simpleHTTP script to assist with theme and site development? **N**
* Do you want to upload your website using ...? **Y for only GitHub Pages**
* Is this your personal page (username.github.io)? **Y**

Ignore the warning regarding the existing `output` directory. You should
now also see a content directory and some configuration files.

Amend the following line in `publishconf.py` to ensure Pelican does
not delete all content in the output directory prior site regeneration
(including your git submodel repository!).

```python
DELETE_OUTPUT_DIRECTORY = False
```

## Creating Your First Article

All content in the Pelican source project should be written in
[Markdown](https://en.wikipedia.org/wiki/Markdown)
or [reStructured Text (rst)](http://www.sphinx-doc.org/en/stable/rest.html). To begin, simply
save the articles as .md or .rst files in the `content` directory (e.g.
`ghpages/content/create-github-page.rst`). As your site grows you can
consider alternate methods for organization.

I decided to use `rst`. You can use your favourite text editor or an IDE
but I would recommend the [Online RestructuredText editor](http://rst.ninjs.org/). You can see the formatted output as you
write which is very helpful. Note that it will look different in the
final site once CSS is applied so don't get worried about font or
anything like that yet.

Make sure to add metadata aligning with [these requirements](http://docs.getpelican.com/en/stable/content.html#file-metadata).

**Update:** After about three months of work with Pelican using
reStructured Text I've settled to using
[PyCharm](https://www.jetbrains.com/help/pycharm) for editing & the
options outlined below for previewing in a web browser. The option I
recommended earlier is okay for quickly previewing how rst syntax works
but is not practical for continuous editing, saving & HTML generation.
To be honest, however, I'm considering switching to markdown due to the
broader built-in support for editing & preview (Atom etc).

## Building & Serving with HTTP

Build the site using the `pelican` utility. I would recommend using the
`-r` or `--autoreload` option which will auto-reload the site allowing
you to edit documents and review changes without restarting pelican.

```shell
cd /path/to/ghpages
pelican content/ -r
```

Open a new shell session in a new terminal and start the Python simple
HTTP server to preview how it will look when hosted by GitHub.

In Python 2:

```shell
cd /path/to/ghpages/output
python -m SimpleHTTPServer
```

In Python 3:

```shell
cd /path/to/ghpages/output
python -m http.server
```

Then navigate to http://localhost:8000/ and look at your site. You can
edit any of your pages as you go and simply refresh your browser to see
the results.

## Adding Social Media Links

You can easily add social media links to your main page by editing the
**pelicanconf.py** file.

For example:

```python
# Social widget
SOCIAL = (('github', 'http://github.com/you'),
          ('linkedin', 'https://linkedin.com/in/you'),
         )
```

Because you edited the .py files, you will need to restart pelican to
see the effect. In the terminal press CTRL+C to kill the process then
the up arrow & enter to run the command again.

You should see the links either at the bottom or side depending on the
site's theme (with the current default, it will be at the bottom).
Included are all the appropriate icons (awesome!).

## publishing

Once you are satisfied you can kill both the HTTP and pelican services
with CTRL+C.

Before publishing, you want to generate the site again while including
the publishconf.py settings. Based on our default set-up this will add
absolute URLs which is important for various add-ons you may later want
such as disqus comments.

```shell
cd /path/to/ghpages
pelican content/ -s publishconf.py
```

Add, commit & push the files in the output git submodule.

```shell
cd output
git add .
git commit -m "First post."
git push -u origin master
```

Once complete, do the same in the source repository.

```shell
cd ..
echo '*.pyc' >> .gitignore #don't need pyc files
git commit -m "First commit."
git push -u origin master
```

Visit your site at http://you.github.io and see the results!

## Next Steps

Before making a second post, you probably want to:

* create an **about page** within `content/pages/about.rst`
* pick one of the [many awesome themes](http://www.pelicanthemes.com/)

In my next post I'll cover installing and applying themes.
