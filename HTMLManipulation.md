# Manipulate HTML Responses

## Architecture layer: node.js proxy

Use when you want to change the HTML returned by the origin server.

ModSecurity is weak when it comes to manipulating the body of the response returned from the origin server. This is in part due to the complexity of manipulate a data stream in a performant manner.

With our own node.js proxy, we can either buffer the full response received from the origin server, or process it in a streaming fashion.

If we process it as a stream, then we are able to take advantage of node's IO model and introduce the least latency.

In the onRequest handler, we check if we need to modify the response - so that we don't run the onResponse handler for every response:

```JavaScript
exports.onRequest = function(options) {
    return function autocompleteOnRequest (proxyRequest, request, response, options) {
        if ((proxyRequest.path.toLowerCase().startsWith("/account/login")) || 
            (proxyRequest.path.toLowerCase().startsWith("/account/register")) ||
        (proxyRequest.path.toLowerCase().startsWith("/manage/changepassword"))) {
            response.autocomplete_active_uri = 1;
        }
    }
}
```

In the onResponse handler we set up a [Harmon](https://github.com/No9/harmon) filter, which runs the HTML received through the [Trumpet](https://github.com/substack/node-trumpet) library. This allows us to parse streaming HTML by using familiar CSS selector syntax.

Harmon calls our function with HTML nodes that match the query, and we can alter the contents or attributes of that node.

In this example, we search for password input fields, and set the `autocomplete=off` attribute.

```JavaScript
exports.onResponse = function (options) {  
    return function autocompleteOnResponse (proxyResponse, request, response, options) {
        if (response.autocomplete_active_uri) {
            var responseSelects = [];
            var simpleselect = {};

            // Find all password fields and add the autocomplete=off attribute
            simpleselect.query = 'input[type=password]';
            simpleselect.func = function (node) {
                node.setAttribute('autocomplete','off');
            }

            responseSelects.push(simpleselect);

            var harmonfilter = harmon([], responseSelects);
            harmonfilter(null, response, function(){});
        }
    }
}
```

To use this patch, you will need to include the Harmon library.