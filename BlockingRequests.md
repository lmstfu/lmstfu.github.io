# Blocking Requests

Sometimes your rule will decide to block a request, redirect the request to a different page, or otherwise avoid sending the request to the original web page.

## Architecture layer: modsecurity

In modsecurity you can specify a *disruptive action* in the list of actions. In the below example, `deny` is the disruptive action:

```ApacheConf
SecRule  REQUEST_FILENAME "@rx /order/details/" \
  "id:11101,phase:1,deny,log,\
  t:none,t:lowercase,t:normalisePath,\
  msg:'Blocking access to %{MATCHED_VAR}'"
```

If the `VARIABLES` and `OPERATOR` match, then the `ACTIONS` clause of the SecRule will run. 

The following disruptive actions are documented in the [Reference Manual](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#actions):

* allow: Stops rule processing on a successful match and allows the transaction to proceed.
* block: Performs the disruptive action defined by the previous SecDefaultAction
* deny: Stops rule processing and intercepts transaction.
* drop: Initiates an immediate close of the TCP connection by sending a FIN packet.
* pass: Continues processing with the next rule in spite of a successful match.
* pause: Pauses transaction processing for the specified number of milliseconds. 
* proxy: Intercepts the current transaction by forwarding the request to another web server using the proxy backend.
* redirect: Intercepts transaction by issuing an external (client-visible) redirection to the given location

The most common way to block a request is to set the `deny` or `block` actions. For maintainability, it is recommended to use `block`, as that allows the site administrator to specify what should happen in the SecDefaultAction.

_or_

## Architecture layer: node.js proxy

The node.js proxy gives us a few different ways to finish a response prematurely.

```javascript
proxyRequest.abort();
```

This aborts the request that was going to be sent to the origin webserver. The client browser will see an unfriendly "ECONNRESET" message in their browser.

```javascript
response.writeHead(302,
    { Location: '/Order?Security_Violation' }
);
response.end();
proxyRequest.abort();
```

This will abort the request being forwarded to the origin server, and also write a redirect response back to the client. This allows you to be slightly more friendly by redirecting to a blocking page.
