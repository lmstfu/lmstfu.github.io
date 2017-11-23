## Node.js Proxy

There is a stable proxy implemented in node called [node-js-proxy](https://github.com/nodejitsu/node-http-proxy). This is used by many people for load balancing node applications.

The [Redbird reverse proxy](https://github.com/OptimalBits/redbird) extends node-http-proxy with additional features that make it a good platform to use for our purposes.

The node proxy works just like any standard proxy. Requests from the browser are turned into proxyRequest objects in your code, and can be manipulated. The proxy then forwards the request to the origin webserver, allows manipulation of the response, and returns the reply to the browser.

Implementing the proxy is fairly straight-forward:

```JavaScript
var redbird = require('redbird');

// require() any handler objects here
var example = require('./example');

var reverseProxy = redbird(
    {
        port:5470
    });

reverseProxy.register("localhost", "http://example.org:5000/", {useTargetHostHeader: true});

// Attach handlers to the proxy
reverseProxy.proxy.on('proxyReq', example.onRequest());
reverseProxy.proxy.on('proxyRes', example.onResponse());

// Optionally, add a handler for errors:
reverseProxy.proxy.on('error', function (err, req, res) {
  res.writeHead(500, {
    'Content-Type': 'text/plain'
  });

  res.end('Something went wrong. And we are reporting a custom error message.');
});
```

The main points of configuration are the `register` call to specify the url of the origin server, and adding each handler to the chain of handlers - first by `require`ing it, and then by adding to the request or response events on the proxy.