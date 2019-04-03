+++
title = "Adding Pelican Plugins"
date = "2016-12-02T18:38:00-05:00"
tags = ["Python", "Pelican", "GitHub"]
slug = "adding-pelican-plugins"
summary = """
Much of the diserable functionality for a modern website is delivered via community extensions for the Pelican package. I provides instructions for certain examples in this post, including the Disqus Comments extension.
"""
+++

## Background

I've already written twice about [Pelican](http://docs.getpelican.com)
and how to use it to create and style your GitHub Personal Page. I'm
going to continue with one more post before moving onto other things.

There's just so much more you can do with it through the smorgasbord of
extensibility contained in the [Pelican plugins
repo](https://github.com/getpelican/pelican-plugins).

The two plugins I'll be covering here are:

* [series](https://github.com/getpelican/pelican-plugins/tree/master/series):
  joins separate articles into a linkable series
* [tag
  cloud](https://github.com/getpelican/pelican-plugins/tree/master/tag_cloud):
  presents article tags as a cloud of links, proportional in size to
  the number of times the tag is used

## Installing

The GitHub readme suggests cloning the repo and adding the path to your
**pelicanconf.py** file. I'm doing something similar in principle but
different in practice. I've made the repo a submodule of my source repo.
This essentially clones it while also making it simple to reference no
matter the device or platform I may be using.

```shell
cd /path/to/source
git submodule add https://github.com/getpelican/pelican-plugins
```

Because the pelican-plugins repo itself is full of submodules, a second
git command is required (there is no `add --recursive`, at least not
yet).

```shell
git submodule update --init --recursive
```

You should now have all the plugins you will ever need in the
`pelican-plugins` sub-directory of your source repo. For more on
working with submodules in your project see: https://git-scm.com/book/en/v2/Git-Tools-Submodules

## Adding Plugins

Update the `pelicanconf.py` file with the following two lines.

```python
PLUGIN_PATHS = ['pelican-plugins']
PLUGINS = ['series', 'tag_cloud']
```

Of course, list only those specific plugins you want. How the plugins
work is dependent on the templates and therefore the theme you have
chosen. If your following along exactly you have `pelican-bootstrap3`
which works nicely with both the `series` and `tag_cloud` plugins.
If the plugin you want doesn't work with the theme you've adopted you
will need to add custom [Jinja2](http://jinja.pocoo.org/docs/dev/)
templates.

### Plugins Example 1 - series

This plugin makes your articles part of a series. How this is presented
is largely dependent on the pelican theme you are using. Currently, I'm
using pelican-bootstrap3 which adds a nice element at the bottom of the
article listing all previous and next articles in the series.

All you need to do is add the `:series:` element to your article
metadata, for example:

```
:series:  Pelican
```

The order is based on the article's `:date:` metadata but can be
overridden using the `:series_index:` metadata element.

### Plugins Example 2 - tag_cloud

This plugin will create a "tag cloud" which provides a means to link
articles by the `:tags:` metadata. The size of each link is presented in
proportion to the number of articles with that tag.

By default in pelican-bootstrap3, the tag cloud is displayed as a list
on the right sidebar.

To benefit simply add a comma-separated list of tags to each article's
metadata, for example:

```
:tags:  python, github, pelican
```

## Next Steps

This will be last article on Pelican for awhile. There is still much to
explore in both creating and visiting Pelican sites but it's time to
move on to covering something else.
