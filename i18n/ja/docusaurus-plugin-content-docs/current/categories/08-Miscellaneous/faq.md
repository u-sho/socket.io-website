---
title: FAQ
sidebar_position: 1
slug: /faq/
---

## Can I use wildcards in events? {#can-i-use-wildcards-in-events}

Not in Socket.IO directly, but check out [this plugin](https://github.com/hden/socketio-wildcard) by Hao-kang Den. It provides a Socket.IO middleware to deal with wildcards.


## Prevent flooding from single connection? {#prevent-flooding-from-single-connection}

Limit number of events by `IP`, `uniqueUserId` or/and `socket.id` with [rate-limiter-flexible](https://github.com/animir/node-rate-limiter-flexible/wiki/Overall-example#websocket-single-connection-prevent-flooding) package.

## Socket.IO with Apache Cordova? {#socketio-with-apache-cordova}

Take a look at [this tutorial](/socket-io-with-apache-cordova/).

## Socket.IO on iOS? {#socketio-on-ios}

Take a look at [socket.io-client-swift](https://github.com/socketio/socket.io-client-swift).

## Socket.IO on Android? {#socketio-on-android}

Take a look at [socket.io-client.java](https://github.com/nkzawa/socket.io-client.java).

## Usage with [express-session](https://www.npmjs.com/package/express-session) {#usage-with-express-session}

```js
const express = require('express');
const session = require('express-session');
const app = express();

const server = require('http').createServer(app);
const io = require('socket.io')(server);

const sessionMiddleware = session({ secret: 'keyboard cat', cookie: { maxAge: 60000 }});
// register middleware in Express
app.use(sessionMiddleware);
// register middleware in Socket.IO
io.use((socket, next) => {
  sessionMiddleware(socket.request, {}, next);
  // sessionMiddleware(socket.request, socket.request.res, next); will not work with websocket-only
  // connections, as 'socket.request.res' will be undefined in that case
});

io.on('connection', (socket) => {
  const session = socket.request.session;
  session.connections++;
  session.save();
});

const port = process.env.PORT || 3000;
server.listen(port, () => console.log('server listening on port ' + port));
```
