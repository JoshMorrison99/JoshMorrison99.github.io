---
title: PortSwigger - Client-side Prototype Pollution in Third Party Libraries
date: 2025-04-14 17:42:00 -5000
categories: [CTF]
tags: [ctf, web, prototype pollution]     # TAG names should always be lowercase
---

<br/>

This lab is vulnerable to DOM XSS via client-side prototype pollution. This is due to a gadget in a third-party library, which is easy to miss due to the minified source code. Although it's technically possible to solve this lab manually, we recommend using DOM Invader as this will save you a considerable amount of time and effort.

<br/>

To solve the lab:
1. Use DOM Invader to identify a prototype pollution and a gadget for DOM XSS.
2. Use the provided exploit server to deliver a payload to the victim that calls `alert(document.cookie)` in their browser.

<br/>

This lab is based on real-world vulnerabilities discovered by PortSwigger Research. For more details, check out Widespread prototype pollution gadgets by Gareth Heyes.

<br/>

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/0.png)
*Website Landing Page*

<br/>

We will be using DOM Invader to solve this task, so make sure you have `Prototype pollution is on`.

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/1.png)

<br/>

After turning on prototype pollution you can see that we have `2 vulnerable sources via prototype pollution in the hash`. A `source` is your input, while a sink is where the input end up in. When DOM invader is saying `2 vulnerable sources via prototype pollution in the hash` it is telling us that `we are able to modify an objects prototype`. Basically, we have prototype pollution and now we need to find a way to exploit it. 

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/2.png)

<br/>

# How did DOM Invader find that an object can be polluted?

1. Go to the setting for prototype pollution.

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/3.png)

<br/>

Click on `Techniques`

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/4.png)

<br/>

This is all the techniques that DOM Invader uses to find prototype pollution.

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/5.png)

<br/>

When you browse to a website, DOM Invader will send a request containing all of these payloads:

```jsx
https://0a4a00c404f1c0b6801a03c400040031.web-security-academy.net/
?constructor[prototype][a42e5579]=tl45dypb
&constructor.prototype.b1a3fd5b=tl45dypb
&__proto__.ccd80966=tl45dypb
&__proto__[dcb52823]=tl45dypb
&constrconstructoructor[prototype][a55a1ee1]=tl45dypb
&constrconstructoructor.prototype.b2f55e1f=tl45dypb
&__pro__proto__to__.eab10255=tl45dypb
&__pro__proto__to__[f33fdea1]=tl45dypb#constructor[prototype][a3aa3232]=tl45dypb
&constructor.prototype.bf1e103d=tl45dypb
&__proto__.c5e2cbce=tl45dypb
&__proto__[d0992d86]=tl45dypb
&constrconstructoructor[prototype][af3a3098]=tl45dypb
&constrconstructoructor.prototype.bac11f2e=tl45dypb
&__pro__proto__to__.e1a3af2f=tl45dypb
&__pro__proto__to__[f122de92]=tl45dypb
```

<br/>

>When you refresh the page with DOM Invader on and with Prototype pollution enabled, you can briefly see the payload being sent.
{: .prompt-info }

<br/>

> Why does there appear to be duplicates? I don’t know…
{: .prompt-info }

<br/>

To check the specify request being sent that is responsible for polluting the prototype, you can click the `Test` button which will open a new tab with the request and a console log of the polluted object. 

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/6.png)

<br/>

The request responsible for polluting the object’s prototype is via the hash: `#**proto**[testproperty]=DOM_INVADER_PP_POC` 

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/7.png)

<br/>

# Finding the Sink

To find this gadget it really isn’t something to do manually as the gadget is a very long block of minified javascript. After the gadget is found click `exploit` and an alert box will pop.

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/8.png)

<br/>

![](/assets/portswigger/client-side-prototype-pollution-in-third-party-libraries/9.png)

<br/>

Once the alert box pops, use the following code in the exploit server to get the victim to pop the alert box on their end.

```jsx
<script>
window.location.replace("https://0a4a00c404f1c0b6801a03c400040031.web-security-academy.net/#__proto__[hitCallback]=alert(document.cookie)")
</script>
```
