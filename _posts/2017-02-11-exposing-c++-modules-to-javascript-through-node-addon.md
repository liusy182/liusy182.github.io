---
layout: post
title:  Exposing c++ modules to JavaScript through node addon
date:   2017-02-11 07:55:20 +0800
categories:
tags:
- Node Addon
- JavaScript
- C++
- CPP
- V8
---

In [`Node`] back end projects or desktop projects using [`Electron`], there are times where we need to access certain native capabilities that node has yet exposed through its JavaScript APIs. In enterprise applications, there are also times where a Node application needs to integrate with a preexisting native module. All these requirements are made possible through [`Node Addon`] that allows JavaScript to harness the power of C/C++ libraries.

There are [good examples](https://github.com/nodejs/node-addon-examples) on how to write simple Node Addons out there in Github. Taking that as a starting point, I was able to integrate an existing C++ module into Node, and exposed its APIs in JavaScript. In this post, I am going to share a few tips I learnt during integration.

The native module that I need to integrate is a sign in module written in C++, lets call it `foo_sign_in.dll`. It is distributed as a dynamically linked library. I find it is most convenient to follow the example of [object wrap]. My file organization looks like this:

{% highlight text %}
/native_app
    index.js - (JavaScript file that loads the node native module and export it)
    binding.gyp - (build configuration file)
    /src
        native_app.cc - (defines program's entry point)
        /api - (api layer that defines a set of exposed JavaScript APIs)
            api_sign_in.cc
            api_sign_in.h
        /internal - (implementation layer that contains the actual initialization / interfacing with the foo_sign_in.dll)
            sign_in.cc
            sign_in.h
{% endhighlight %}


in `binding.gyp`, we need to load the .lib file and specify a list of files to build by following [gyp's documentation](https://gyp.gsrc.io/docs/UserDocumentation.md):


{% highlight python %}
#`binding.gyp`:
{
  'variables': {
    #...
  },
  'targets': [
    {
      'target_name': 'native_app',
      'msvs_guid': '',
      'sources': [
        'src/native_app.cc',
        'src/api/api_sign_in.cc',
        'src/internal/sign_in.cc',
      ],
      'dependencies': [
        #...
      ],
      'defines': [
        #...
      ],
      'include_dirs': [
        #...
      ],
      'libraries': [
        #...
      ],
      'conditions': [
        ['OS=="win"', {
          'sources': [
            #...
          ],
          'defines': [
            #...
          ],
          'include_dirs': [
            #...
          ],
          'libraries': [
            '<!(echo %INCLUDEDIR%)/foo_sign_in.lib'
          ]
        }]
      ],
    },
  ],
}
{% endhighlight %}

I have removed a few my project-specific settings. But you get the idea. There are a few things to take note here. First is that gyp allows platform-specific build configurations to be easily separated. Since `foo_sign_in` is a Windows module, I have kept linking of `foo_sign_in.lib` in Windows branch under condition `OS=="win"`. The other thing is that `INCLUDEDIR` is a environmental variable that can be set to any directory from command line. The syntax `<!(echo %VARIABLE_NAME%)` is gyp's way of expanding a variable.

`internal/sign_in` contains code to interface with `foo_sign_in` module. In this simplified example, the class only has 2 methods `signIn` and `signOut`. The header file `sign_in.h` looks something like this:

{% highlight cpp %}
//sign_in.h:

#ifndef _INTERNAL_SIGN_IN_H_
#define _INTERNAL_SIGN_IN_H_
#include "foo_sign_in.h" // header from module foo_sign_in
#include <string>

namespace nativeApp {

class SignIn {

public:
  SignIn();
  ~SignIn();

  bool signIn();
  bool signOut();
};

}
#endif

{% endhighlight %}

The implementation in `sign_in.cc` is just a wrapper on top of `foo_sign_in`:

{% highlight cpp %}
//sign_in.cc:

#include "sign_in.h"

namespace nativeApp {

SignIn::SignIn() {
  FooSignIn::Initialize();
}

SignIn::~SignIn() {
}

bool SignIn::signIn() {
  return FooSignIn::GetInstance()->signIn();
}

bool SignIn::signOut() {
  return FooSignIn::GetInstance()->signOut();
}

} 

{% endhighlight %}

Having this layer of wrapper has a few advantages that are not obvious in this simple example. One of the benefits is that complexity is encapsulated if integration with the external module is not as simple as calling the API but involves some amount of custom logic. The other benefit is that a single API is exposed although underneath there could be totally different implementations in different platforms in future.

In the API layer, we extend from the object `node::ObjectWrap` and keep `SignIn` object as a private variable. Basically, object `ApiSignIn` becomes a wrapper on top of object `SignIn`:

{% highlight cpp %}
//api_sign_in.h:

#ifndef _API_SIGN_IN_H_
#define _API_SIGN_IN_H_

#include <node.h>
#include <node_object_wrap.h>
#include <memory>
#include "internal/sign_in.h"

namespace nativeApp {

class ApiSignIn : public node::ObjectWrap {
 public:
  static void Initialize();
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);
 private:
  ApiSignIn();
  ~ApiSignIn();
  static v8::Persistent<v8::Function> constructor;
  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);

  static void SignIn(const v8::FunctionCallbackInfo<v8::Value>& args);
  static void SignOut(const v8::FunctionCallbackInfo<v8::Value>& args);

  // Wrapped object, keep as a private var
  std::unique_ptr<SignIn> w_;
};

} 

#endif

{% endhighlight %}

In `api_sign_in.cc`, we let `SignIn` and `SignOut` returns a callback function with argument boolean indicating whether sign in / out is successful:

{% highlight cpp %}
//api_sign_in.cc:

#include <node.h>
#include "api_sign_in.h"

using namespace v8;

namespace nativeApp {

Persistent<Function> ApiSignIn::constructor;

ApiSignIn::ApiSignIn(const FunctionCallbackInfo<Value>& args) {
  w_.reset(new SignIn();
}

ApiSignIn::~ApiSignIn() {}

void ApiSignIn::Initialize(Handle<Object> exports) {
  // Prepare constructor template
  Isolate* isolate = Isolate::GetCurrent();
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
  tpl->SetClassName(String::NewFromUtf8(isolate, "ApiSignIn"));
  tpl->InstanceTemplate()->SetInternalFieldCount(1);  

  NODE_SET_PROTOTYPE_METHOD(tpl, "signIn", SignIn);
  NODE_SET_PROTOTYPE_METHOD(tpl, "signOut", SignOut);

  constructor.Reset(isolate, tpl->GetFunction());
  exports->Set(String::NewFromUtf8(isolate, "ApiSignIn"), 
              tpl->GetFunction());
}

void ApiSignIn::New(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = Isolate::GetCurrent();
  HandleScope scope(isolate);

  ApiSignIn* w = new ApiSignIn(args);
  w->Wrap(args.This());
  args.GetReturnValue().Set(args.This());
}

void ApiSignIn::SignIn(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = Isolate::GetCurrent();
  HandleScope scope(isolate);

  ApiSignIn* obj = ObjectWrap::Unwrap<ApiSignIn>(args.Holder());
  bool ret = obj->w_->signIn();

  Local<Function> cb = Local<Function>::Cast(args[0]);
  const unsigned argc = 1;
  Local<Value> argv[argc] = { Boolean::New(isolate, ret) };
  cb->Call(isolate->GetCurrentContext()->Global(), argc, argv);
}

void ApiSignIn::SignOut(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = Isolate::GetCurrent();
  HandleScope scope(isolate);

  ApiSignIn* obj = ObjectWrap::Unwrap<ApiSignIn>(args.Holder());
  bool ret = obj->w_->signOut();

  Local<Function> cb = Local<Function>::Cast(args[0]);
  const unsigned argc = 1;
  Local<Value> argv[argc] = { Boolean::New(isolate, ret) };
  cb->Call(isolate->GetCurrentContext()->Global(), argc, argv);
  
}


} 

{% endhighlight %}

Then, we need to export and initialize `ApiSignIn` in the entry point `native_app.cc`:

{% highlight cpp %}
//native_app.cc:

#include <node.h>
#include "api/api_sign_in.h"
using namespace v8;

namespace nativeApp {

void Initialize(Handle<Object> exports) {
  ApiSignIn::Initialize(exports);
}

NODE_MODULE(nativeApp, Initialize)
}

{% endhighlight %}

Last but not least, we need to require the native module and expose it to the rest of the JavaScript program. In this case, I am exposing `ApiSignIn` as a singleton instance in `native_app/index.js`:

{% highlight JavaScript %}
//index.js:

'use strict';

let _nativeApp;
try {
  _nativeApp = require('./build/Release/nativeApp.node');
} catch (err) {
  _nativeApp = require('./build/Debug/nativeApp.node');
}

class NodeSignIn {
  constructor () {
    this.nodeSignIn = new _nativeApp.ApiSignIn();
  },

  signIn(cb) {
    this.nodeSignIn.signIn(cb);
  }

  signOut(cb) {
    this.nodeSignIn.signOut(cb);
  }
}

module.exports = new NodeSignIn();

{% endhighlight %}

If a promise object is preferred, we can also wrap `NodeSignIn`'s `signIn` and `signOut` and return promises instead.

This concludes the example. Essentially, we created a chain of wrappers (JavaScript wraps node API which wraps c++ implementation which wraps the native dll) to expose a native dll to Node runtime. This chain of wrappers makes responsibilities clear and program easy to understand, and most importantly, friends are made between ancient C++ and modern JavaScript! 

[`Node`]: https://github.com/nodejs/node
[`Electron`]: https://github.com/electron/electron
[`Node Addon`]: https://nodejs.org/api/addons.html
[object wrap]: https://github.com/nodejs/node-addon-examples/tree/master/6_object_wrap/node_0.12