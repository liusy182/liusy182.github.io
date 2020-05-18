---
layout: post
title:  Handle native events in node addon
date:   2017-02-19 07:55:20 +0800
categories:
tags:
- Node Addon
- JavaScript
- C++
- CPP
- V8
---

Following up [`the post`] on node addon, one of another use cases in node addon is that events coming from native modules needs to be emitted to JavaScript client. On JavasScript side, the common way to achieve this is to pass a callback function to the native. The callback is expected to be invoked when the event is triggered. By default, v8 local variables are garbage collected, which means that callback function that is passed in as function argument is not persisted. v8 provides a `Persistent<T>` template to cater for this kind of use case. By declaring an object as `Persistent<T>`, the object will live as long as the program is still running, until it is explicitly dismissed.

The C++ native sign in library that I need to integrate emits a `loggedIn` event when sign in activity happens. JavaScript client is interested in knowing this event. On the JavasScript layer, we pass a callback to native module:

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
    this.handlers = [];
    //pass in a callback to listen to `loggedIn` event
    this.nodeSignIn.loggedIn(() => this.handlers.map((cb) =>  cb()));
  },

  onLoggedIn: function(cb) {
    this.handlers.push(cb);
  }
}

module.exports = new NodeSignIn();

{% endhighlight %}

On the node addon API layer, we use `Persistent<T>` to store the callback coming from JavaScript. The code below is taken from my previous [`post`], with new APIs added.

{% highlight cpp %}
//api_sign_in.h:

#ifndef _API_SIGN_IN_H_
#define _API_SIGN_IN_H_

#include <node.h>
#include <node_object_wrap.h>
#include <memory>
#include "internal/sign_in.h"

namespace nativeApp {

class ApiSignIn : 
public node::ObjectWrap
public ISignInObserver  {
 public:
  static void Initialize();
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);

  //from ISignInObserver
  virtual void loggedIn() override;
 private:
  ApiSignIn();
  ~ApiSignIn();
  static v8::Persistent<v8::Function> constructor;
  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);

  static void SignIn(const v8::FunctionCallbackInfo<v8::Value>& args);
  static void SignOut(const v8::FunctionCallbackInfo<v8::Value>& args);

  // persistent storage
  static v8::Persistent<v8::Function> loggedInCallback_;

  // Wrapped object, keep as a private var
  std::unique_ptr<SignIn> w_;
};

} 

#endif

{% endhighlight %}

Note that this class is derived from `ISignInObserver`, following observer pattern. `ISignInObserver` comes from the C++ library that I am integrating. The class `ApiSignIn` will get notified when `ISignInObserver::loggedIn` happens. What we need to do next is to store JS callback function in `loggedInCallback_`, and invoke this callback when loggedIn event happens. Again, this code is taken from my previous [`post`] with a few changes:

{% highlight cpp %}
//api_sign_in.cc:

#include <node.h>
#include "api_sign_in.h"

using namespace v8;

namespace nativeApp {

Persistent<Function> ApiSignIn::constructor;
Persistent<Function> ApiWebServiceClient::loggedInCallback_;

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
  // expose `loggedIn(cb)` to JavaScript
  NODE_SET_PROTOTYPE_METHOD(tpl, "loggedIn", LoggedIn);

  constructor.Reset(isolate, tpl->GetFunction());
  exports->Set(String::NewFromUtf8(isolate, "ApiSignIn"), 
              tpl->GetFunction());
}

void ApiSignIn::New(const FunctionCallbackInfo<Value>& args) {
  //...
}

void ApiSignIn::SignIn(const FunctionCallbackInfo<Value>& args) {
  //...
}

void ApiSignIn::SignOut(const FunctionCallbackInfo<Value>& args) {
  //...
}

//ISignInObserver::loggedIn 
void ApiSignIn::loggedIn() {
  if(loggedInCallback_.IsEmpty()) return;
  Isolate* isolate = Isolate::GetCurrent();
  Local<Function> func = Local<Function>::New(isolate, loggedInCallback_);
  if(func.IsEmpty()) return;
  const unsigned argc = 1;
  Local<Value> argv[argc] = { Undefined(isolate) };
  func->Call(Null(isolate), argc, argv);
}

void ApiSignIn::LoggedIn(const v8::FunctionCallbackInfo<v8::Value>& args) {
  Isolate* isolate = Isolate::GetCurrent();
  loggedInCallback_.Reset(isolate, Local<Function>::Cast(args[0]));
}

} 

{% endhighlight %}

The magic happens in `ApiSignIn::loggedIn` and `ApiSignIn::LoggedIn` (note the difference in casing). In `ApiSignIn::LoggedIn`, we store the callback function to `Persistent<Function> loggedInCallback_` by using the `Reset` API. When the observer `ApiSignIn::loggedIn` is triggered, we will retrieve `loggedInCallback_`, casting it to a `Local<Function>`, and invoke it. This simple example illustrates how v8's `Persistent<T>` API can be used in event handlers.

[`Node`]: https://github.com/nodejs/node
[`Electron`]: https://github.com/electron/electron
[`post`]: /2017/02/10/exposing-c++-modules-to-javascript-through-node-addon.html
[`the post`]: /2017/02/10/exposing-c++-modules-to-javascript-through-node-addon.html
[object wrap]: https://github.com/nodejs/node-addon-examples/tree/master/6_object_wrap/node_0.12