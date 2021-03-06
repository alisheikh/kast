# kast [![Build Status](https://travis-ci.org/akoenig/kast.svg)](https://travis-ci.org/akoenig/kast)

An UDP multicast framework.

## Usage example

Let's say we want to start two `kast` servers, each of which provides a command to check if the respective host is alive.

### Host A

```javascript
var kast = require('kast');

var server = kast();

server.command('/alive', function (req, res) {
    res.send('Hey! Host A is alive!');
});

server.listen(5000);
```

### Host B

```javascript
var kast = require('kast');

var server = kast();

server.command('/alive', function (req, res) {
    res.send('This is Host B speaking! How can I help you?');
});

server.listen(5000);
```

### Client

A client can now send a _broadcast request_ to all the hosts without knowing each individual ip address / host name.

```javascript
var kast = require('kast');

kast.broadcast({
    port: 5000,
    command: '/alive',
}, function onResults (err, results) {
    console.log(results);
});
```

The results of the hosts which have responded in a timely manner will be collected in the `results` argument, like:

```javascript
{
    '10.0.0.1': 'Hey! Host A is alive!',
    '10.0.0.23': 'This is Host B speaking! How can I help you?'
}
```

## API

### Install

```sh
$ npm install --save kast
```

### Basic usage

```javascript

var kast = require('kast');

var server = kast();
```

### Creating a server instance

#### server.listen(port, [host], [callback])

Binds and listens for new connections on the given host and post. Please note that the `host` parameter is optional. `kast` uses `224.1.1.1` as a default multicast group.

The `callback` function will be executed as `callback(err)` after the binding process has been finished.

This method will return an instance of `Socket` which exposes the following API:

##### socket.close(callback)

Possibility for shutting the whole `kast` server down.

### Registering commands

#### server.command(name, handlerFn)

The `handlerFn` will receive two ordinary parameters: A `req`uest and a `res`ponse parameter. Pretty much like [Express](http://expressjs.com/) - You're welcome :).

##### Request

An object with the following information:

  * `req.id`: An unique request id (will be generated by `kast`).
  * `req.connection.remoteAddress`: The ip address of the host which has sent the request.
  * `req.connection.remotePort`: The remote port of the host which has sent the request.
  * `req.command`: The `name` of the command.
  * `req.body`: A possible request body with additional data (comparable with a HTTP request body).

##### Response

While the `req` object provides pure information without any kind of functionality, the `res` object exposes a method with which you can _answer_ a command and send the result back to the client.

###### res.send(body)

Sends a response with the given `body` as string.

### Send a broadcast to the servers

#### kast.broadcast(options, callback)

The broadcast method will usually be called by a client which wants to execute several commands on the respective hosts (within the multicast group).

Possible options are:

  * `port`: The port on which the other hosts are listening.
  * `command`: The name of the command which should be executed.
  * `body`: (optional) Additional data that should be passed to the command.
  * `timeout: (optional; default=2000ms) The timeout after which the broadcast should stop waiting for responses.
  * `host`: (optional; default='224.1.1.1') The ip address of the multicast group.

The callback function will be executed as `callback(err, results)`. `results` will be an object with the ip addresses of the hosts as `keys` and the respective responses as `values`.

## Development

### Tests

In order to run the test suite, install the dependencies first, then run `npm test`:

```sh
$ npm install
$ npm test
```

## License

MIT © 2014, [André König](http://andrekoenig.info) (andre.koenig@posteo.de)
