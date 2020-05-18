---
layout: post
title:  Upgrade to webpack 2
date:   2017-03-28 20:00:20 +0800
categories:
tags:
- webpack
- webpack2
- javascript
- frontend
---

`webpack 2` is out. It didn't take very long for me to realize such tremendous improvements it has made as compared to version 1. Back in the days when version 1 prevailed, I felt so much inertia whenever I needed to update webpack configurations. I was intimidated by both its arcane syntax and documentation. With webpack 2, [documentation] has become crystal clear. Syntax has become much more appealing. I especially like the syntax changes highlighted in its [migration document]. Finally, everything I dislike about webpack is gone -) In this post, I am sharing a few experiences when migrating a project from webpack 1 to webpack 2.

### loaders in general

In version 1, a typical loader would look like

{% highlight JavaScript %}
//...webpack v1
{
  test   : /\.scss$/,
  loaders: ['style', 'css?-mergeRules', 'sass?sourceMap']
}
//...
{% endhighlight %}

I am sure that without any prior knowledge, no one could immediately figure out that 'style' means 'style-loader' and 'css?-mergeRules' means 'css-loader with mergeRules disabled' in the above code. Things become much more intuitive in version 2 where loaders' full name become compulsory, and nested objects are used to denote different options.

{% highlight JavaScript %}
//...webpack v2
{
  test: /\.scss$/,
  use: [{
    loader: 'style-loader'
  }, {
    loader: 'css-loader',
    options: { mergeRules: false }
  }, {
    loader: 'sass-loader', 
    options: { sourceMap: true }
  }]
}
//...
{% endhighlight %}

### documentation and export-loader

As of today, documentation still needs to catch up a bit as there are places where things are sparsely documented or simply marked as 'TODO'. I had to give a glimpse on [`export-loader`]'s code to figure out how to migrate version 1 syntax to version 2.

{% highlight JavaScript %}
//...webpack v1
{
  test: /external.js$/,
  query: "exportedObject",
  loader: 'exports'
}
//...
{% endhighlight %}

v.s.

{% highlight JavaScript %}
//...webpack v2
{
  test: /external.js$/,
  use: [{
    loader: 'exports-loader',
    options: { 'exportedObject': true }
  }]
}
//...
{% endhighlight %}

### export function in webpack.config.js

One great feature I like in the new version is that `webpack.config.js` now allows exporting a function instead of an object. E.g.

{% highlight JavaScript %}
//webpack.config.js
module.exports = function buildConfig(env) {
  return require('./config/' + env + '.js')(env)
}
{% endhighlight %}

The value of `env` is passed through command prompt when building, e.g.

{% highlight text %}
webpack --env=dev --progress --profile --colors
{% endhighlight %}

This means we can separate configurations for different environments into separate files easily. Just like before, everything can still inherit from a base configuration using [`webpack-merge`]. Details are highlighted in [its documentation](https://webpack.js.org/guides/production-build/#the-manual-way-configuring-webpack-for-multiple-environments)


[documentation]: https://webpack.js.org/api/cli/

[migration document]: https://webpack.js.org/guides/migrating/

[`export-loader`]: https://github.com/webpack-contrib/exports-loader

[`webpack-merge`]: https://github.com/survivejs/webpack-merge