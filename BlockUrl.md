# Block access to a url

## Architecture layer: modsecurity

Use when you want to block access to a single url on the site.

```ApacheConf
SecRule  REQUEST_FILENAME "/order/details/" \
  "id:11101,phase:1,deny,log,\
  t:none,t:lowercase,t:normalisePath,\
  msg:'Blocking access to %{MATCHED_VAR}'"
```

This rule lowercases the url and compares it to "/order/details/". If it matches then it is denied.

You will need to update the ID to a number not already in use, and you may want to modify the message.

_or_

## Architecture layer: node.js proxy

URL blocking can also be performed in the node.js proxy, if you need to perform more actions in order to make your decision.

```javascript
exports.onRequest = function(options) {
    return function orderSlashOnRequest (proxyRequest, request, response, options) {
        if (proxyRequest.path.toLowerCase().startsWith("/order/details/")) {
            proxyRequest.abort();
        }
    }
}
```
