---
title: Admin UI
sidebar_position: 3
slug: /admin-ui/
---

The Socket.IO admin UI can be used to have an overview of the state of your Socket.IO deployment.

The source code can be found here: https://github.com/socketio/socket.io-admin-ui/

Link to the hosted version: https://admin.socket.io/

## Current features {#current-features}

- overview of the servers and the clients that are currently connected

![dashboard screenshot](/images/admin-ui-dashboard.png)

- details of each socket instance (active transport, handshake, rooms, ...)

![socket details screenshot](/images/admin-ui-socket-details.png)

- details of each room

![room details screenshot](/images/admin-ui-room-details.png)

- administrative operations (join, leave, disconnect)

If you have any feedback / suggestions, do not hesitate!

## Installation {#installation}

### Server-side {#server-side}

First, install the `@socket.io/admin-ui` package:

```
npm i @socket.io/admin-ui
```

And then invoke the `instrument` method on your Socket.IO server:

```js
const { createServer } = require("http");
const { Server } = require("socket.io");
const { instrument } = require("@socket.io/admin-ui");

const httpServer = createServer();

const io = new Server(httpServer, {
  cors: {
    origin: ["https://admin.socket.io"],
    credentials: true
  }
});

instrument(io, {
  auth: false
});

httpServer.listen(3000);
```

The module is compatible with:

- Socket.IO v4 server
- Socket.IO v3 server (>= 3.1.0), but without the operations on rooms (join, leave, disconnection)

### Client-side {#client-side}

You can then head up to https://admin.socket.io, or host the files found in the `ui/dist` folder [here](https://github.com/socketio/socket.io-admin-ui/tree/main/ui/dist).

**Important note**: the website at https://admin.socket.io is totally static (hosted on [Vercel](https://vercel.com)), we do not (and will never) store any information about yourself or your browser (no tracking, no analytics, ...). That being said, hosting the files yourself is totally fine.

You should see the following modal:

![login modal screenshot](/images/admin-ui-login-modal.png)

Please enter the URL of your server, including the namespace (for example, `http://localhost:3000/admin` or `https://example.com/admin`) and the credentials, if applicable (see the `auth` option [below](#auth)).

### Available options {#available-options}

#### `auth` {#auth}

Default value: `-`

This option is mandatory. You can either disable authentication (please use with caution):

```js
instrument(io, {
  auth: false
});
```

Or use basic authentication:

```js
instrument(io, {
  auth: {
    type: "basic",
    username: "admin",
    password: "$2b$10$heqvAkYMez.Va6Et2uXInOnkCT6/uQj1brkrbyG3LpopDklcq7ZOS" // "changeit" encrypted with bcrypt
  },
});
```

WARNING! Please note that the `bcrypt` package does not currently support hashes starting with the `$2y$` prefix, which is used by some BCrypt implementations (for example https://bcrypt-generator.com/ or https://www.bcrypt.fr/). You can check the validity of the hash with:

```
$ node
> require("bcrypt").compareSync("<the password>", "<the hash>")
true
```

You can generate a valid hash with:

```
$ node
> require("bcrypt").hashSync("changeit", 10)
'$2b$10$LQUE...'
```

See also:

- https://github.com/kelektiv/node.bcrypt.js/issues/849
- https://stackoverflow.com/a/36225192/5138796

#### `namespaceName` {#namespacename}

Default value: `/admin`

The name of the namespace which will be created to handle the administrative tasks.

```js
instrument(io, {
  namespaceName: "/custom"
});
```

This namespace is a classic Socket.IO namespace, you can access it with:

```js
const adminNamespace = io.of("/admin");
```

More information [here](namespaces.md).

#### `readonly` {#readonly}

Default value: `false`

Whether to put the admin UI in read-only mode (no join, leave or disconnect allowed).

```js
instrument(io, {
  readonly: true
});
```

#### `serverId` {#serverid}

Default value: `require("os").hostname()`

The ID of the given server. If you have several Socket.IO servers on the same machine, you'll need to give them a distinct ID:

```js
instrument(io, {
  serverId: `${require("os").hostname()}#${process.pid}`
});
```

#### `store` {#store}

Default value: `new InMemoryStore()`

The store is used to store the session IDs so the user do not have to retype the credentials upon reconnection.

If you use basic authentication in a multi-server setup, you should provide a custom store:

```js
const { instrument, RedisStore } = require("@socket.io/admin-ui");

instrument(io, {
  store: new RedisStore(redisClient)
});
```

## How it works {#how-it-works}

The source code can be found here: https://github.com/socketio/socket.io-admin-ui/

The `instrument` method simply:

- creates a [namespace](namespaces.md) and adds an authentication [middleware](../02-Server/middlewares.md) if applicable
- register listeners for the `connection` and `disconnect` events for each existing namespaces to track the socket instances
- register a timer which will periodically send stats from the server to the UI
- register handlers for the `join`, `leave` and `_disconnect` commands sent from the UI
