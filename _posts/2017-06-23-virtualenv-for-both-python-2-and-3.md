---
layout: post
title:  Setup virtualenv for both python 2 and 3 on Mac
date:   2017-06-23 10:00:20 +0800
categories:
tags:
- virtualenv
- python
---

It is fairly common to keep 2 versions of python coexist on the same system. Given the popularity of the [`virtualenv`], it would also come in very handy if we can simply run `virtualenv venv` to setup virtual environment for both python 2 and python 3. In this post, I am describing 2 ways to configure `virtualenv` for both versions of python on Mac.

## HomeBrew install

If both python 2 and python 3 are installed through [`HomeBrew`], they most likely reside in the same directory `/usr/local/bin` with different executable names `python` and `python3`. We should see `pip` and `pip3` under the same directory as well. 

We will install `virtualenv` through `pip`. This makes `virtualenv` refer to python 2 as the default environment:

{% highlight Bash %}
pip install virtualenv
{% endhighlight %}

Then, we are going to add an alias in `$HOME/.bash_profile` to refer to python 3 environment:

{% highlight Bash %}
# $HOME/.bash_profile (assume python 3.6 is installed):
alias virtualenv3='virtualenv --python=python3.6'
{% endhighlight %}

Save `.bash_profile`, and run `source .bash_profile` to make changes take effect. After that, running `virtualenv venv` and `virtualenv3 venv` will set up environment for python 2 and 3 respectively. This approach shares the same copy of virtualenv.

## Official package install

If both pythons are installed through official website, they are located under 2 different paths. For example, python 2's default location is `/Library/Frameworks/Python.framework/Versions/2.7/bin/python` and python 3's is `/Library/Frameworks/Python.framework/Versions/3.6/bin/python3`. Similarly, `pip` and `pip3` are under these 2 separate paths. We will install one copy of `virtualenv` under each path:

{% highlight Bash %}
# install under python 2:
pip install virtualenv

# install under python 3:
pip3 install virtualenv
{% endhighlight %}

After that, we will create 2 aliases to refer to each of them:

{% highlight Bash %}
# $HOME/.bash_profile:
alias virtualenv2='~/Library/Python/2.7/bin/virtualenv'
alias virtualenv3='~/Library/Python/3.6/bin/virtualenv'
{% endhighlight %}

2 handy commands get exposed - `virtualenv2` and `virtualenv3` that refer to the underlining `virtualenv` package under different paths.


[`virtualenv`]: https://virtualenv.pypa.io/en/stable/
[`HomeBrew`]: https://brew.sh/

