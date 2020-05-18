---
layout: post
title:  Configure GYP to dynamic CRT in Windows
date:   2017-02-23 20:00:20 +0800
categories:
tags:
- node-gyp
- gyp
- Windows
- CRT
- Run-Time Library
---

[`node-gyp`] is a powerful build tool. The only frustration point is that the documentation for [`gyp`] is very general and lacks a lot of details. Creating a proper project-specific gyp configuration from scratch is not an easy task, which is probably why `node-gyp` project keeps a list of gyp examples [out in the wild] for reference. 

I recently encountered an annoying build error while trying to link a third party library into a project:

{% highlight text %}
LNK2038: mismatch detected for 'RuntimeLibrary': value 'MTd_StaticDebug' doesn't match value 'MDd_DynamicDebug' in program.obj    ...
{% endhighlight %}

After some digging, I got to realize that by default node-gyp uses static version of CRT (C Run-Time library). In Windows system, this means that the project is not built with [/MD flag] set. [This post](http://www.davidlenihan.com/2008/01/choosing_the_correct_cc_runtim.html) and [this post](http://stackoverflow.com/questions/14714877/mismatch-detected-for-runtimelibrary) clearly explain the differences of these configurations.

The next question how this `/MD` flag can be configured into GYP. It turns out that GYP has a key `msvs_settings` for visual studio specific settings. In particular, `/MD` flag is transformed into a number under sub key `VCCLCompilerTool -> RuntimeLibrary`:

{% highlight python %}
#`binding.gyp`:
{
  #...
  'targets': [{
    #...
    'configurations': {
      'Debug': {
        'msvs_settings': {
          'VCCLCompilerTool': {
            'RuntimeLibrary': '3' # /MDd
          }
        }
      },
      'Release': {
        'msvs_settings': {
          'VCCLCompilerTool': {
            'RuntimeLibrary': '2' # /MD
          }
        }
      }
    }
  }]
}
{% endhighlight %}

This configuration is absolutely strange, but well, somehow it does the work :)

[`node-gyp`]: https://github.com/nodejs/node-gyp
[`gyp`]: https://gyp.gsrc.io/docs/UserDocumentation.md
[out in the wild]: https://github.com/nodejs/node-gyp/wiki/%22binding.gyp%22-files-out-in-the-wild
[/MD flag]: https://msdn.microsoft.com/en-us/library/2kzt1wy3(v=vs.71).aspx