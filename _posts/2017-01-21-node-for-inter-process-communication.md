---
layout: post
title:  "Node for inter-process communication"
date:   2017-01-21 12:21:20 +0800
categories:
tags:
- Node
- IPC
- Electron
---

One of the most attractive aspects of [`Node`][Node] is that it does not restrict itself as a web server. Node is also a handy tool when it comes to developing cross-platform desktop applications. [`Electron`][Electron] team has done a great job integrating Node as a dll into their platform. This means that the entire Node API can be made available in a desktop app.

I was very surprised to find that Node fully supports IPC communication from its [`net`][net] module. Node wraps on top of [`libuv`][libuv] and exposes its IPC capability in JavaScript. This is something really cool! I wanted to migrate a project I have been working on from a CEF-hosted browser to Electron framework for a looonggg time. The only bottleneck was to find replacement for a native IPC component written QT's [signal / slot][signal / slot]. IPC communication itself can be relatively easily replicated in Windows named pipes APIs. The difficult part is to get rid of QT's signal / slot. Either the event / callback mechanism has to be redesigned from ground up in C++, or the architecture of the app needs to change. Now with Node, replacing signal / slot is just so straightforward since event / callback is the daily routine in JavaScript :)

Here is a sample code on how an IPC communication can be initiated in Node:

{% highlight JavaScript %}
'use strict';

var net = require('net');

class IPC {
  constructor(pipeName) {
    this.pipeName = pipeName;
    this.client = null;
  }

  connect(cb) {
    this.client = net.connect("\\\\.\\pipe\\" + this.pipeName, 
      () => console.log('connected to ', this.pipeName, '\n');
      cb && cb();
    );

    this.client.on('data', 
      (data) => console.log('data received ', data, '\n');
    );

    this.client.on('end', 
      () => console.log('end connection to ', this.pipeName, '\n');
    );
  }

  write(json) {
    if(!this.client) throw new Error('this client is not initialized');
    
    this.client.write(JSON.stringify(json), 
      () => console.log(this.pipeName, ' write success', '\n');
    );
  }

  disconnect() {
    this.client && this.client.end();
  }
}

// instantiate a named pipe
let ipcTest = new IPC('testPipeName');
ipcTest.connect(() => {
  ipcTest.write ({ foo: bar });
});

setTimeout(() => ipcTest.disconnect(), 10000);


{% endhighlight %}

[Node]: https://github.com/nodejs/node
[Electron]: https://github.com/electron/electron
[QT]: https://www.qt.io/
[net]: https://nodejs.org/api/net.html
[libuv]: https://github.com/libuv/libuv
[signal / slot]: http://doc.qt.io/qt-4.8/signalsandslots.html