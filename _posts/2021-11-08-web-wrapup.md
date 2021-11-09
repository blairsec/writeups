---
layout: post
title:  "web wrapup"
date:   2021-11-08
categories: web
---

Last meeting, we did a bunch of random small web topics, and there were challenges on type juggling, SSRF, and SSTI.

The lecture can be accessed [here](https://docs.google.com/presentation/d/1PC1lSPYNwpGIrL9Dq4WbPv_qGyYdOiLgeJawlKyp13w/edit?usp=sharing).

Challenges
1. [md5 type juggling](#md5-type-juggling)
2. [ssti + ssrf](#ssrf--ssti)

#### md5 type juggling
This challenge was actually a CTF challenge that one of our alums, William Wang, wrote for [ångstromCTF 2018](https://angstromctf.com/)!

The source code is pretty simple:
```php
include 'secret.php';
if($_GET["str1"] and $_GET["str2"]) {
  if ($_GET["str1"] !== $_GET["str2"] and
      hash("md5", $salt . $_GET["str1"]) === hash("md5", $salt . $_GET["str2"])) {
    echo $flag;
  } else {
    echo "Sorry, you're wrong.";
  }
  exit();
}
```

Basically, it takes two query string arguments, `str1` and `str2`, and compares their values and some hashes.

We only get the flag if the two query string arguments differ, but have the same hash when appended to a [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)), and hashed.

The solution is to pass in [query string arrays](https://stackoverflow.com/a/9547490).

The key observation here is to coerce the array into PHP's wack string representation of an array, which is the string `"Array"`. This is done when we concatenate (the `.` operator) the two different arrays with a common salt. This gives the same two hashes, and we get the flag.

This can be seen with the following example:
```php
echo "my_salt" . ["A"];
echo "my_salt" . ["B"];
```

Even though `["A"]` and `["B"]` are clearly different arrays, the output will be:
```
Warning: Array to string conversion in /in/cUqur on line 2
my_saltArray
Warning: Array to string conversion in /in/cUqur on line 3
my_saltArray
```

In the challenge code, `my_saltArray` gets passed into the md5 function twice and of course, the output is the same.

Flag: `flag{does_md5_have_charm}`


#### ssrf + ssti
[This challenge](https://github.com/blairsec/club-challs-2021/tree/master/ssti/site) allows us to ping a server, and write notes (to ourselves, probably).

We also are given that some server that handles templating can be accessed at `template.site`.

##### ssrf
Looking at the code, it seems like the server doesn't actually handle the creation of any templates. Instead, it is passed to another service running on port `8081` internally. So we don't have access to this.

When we create a note, our input is first filtered to check for `{}` characters, which is a pretty good indicator it might have an SSTI vulnerability, which we can later use to take over the server. Then, it is passed on to the service running internally on port `8081`. More specifically, it sends a GET request to `/add` and passes the parameters `content=<filtered_input>`.

So, the server will send a request like `http://localhost:8081/add?content=aaaa`, if `aaaa` were our input.

Since we have a way to make the server send whatever requests we want, let's try to mimic that and create a note, without having our input filtered. We can use `template.site` as our templating server address, and send a request to `/add?content=aaaa`. However, we run into an error: we cannot have `?` in our submitted URL.

The way to get around this is to use a link shortener that redirects to our desired URL. When the server visits our shortened link, it is redirected to the correct URL, but no `?` will appear in our shortened URL. I used tinyurl.

Now we can create a note with whatever content we want, by creating a shortened link that redirects to `http://template.site/add?content=<input>` and submitting it to the server.

##### ssti
With our unlimited note content, we can create a test SSTI payload and submit it. We know the server uses the Jinja 2 templating engine, so all we have to try is `{% raw %}{% endraw %}` as our payload, and submit it with our SSRF method. The long string we get back is the template id, and we can plug it into `/t/<id>` on the website to view our content. It shows `14`, which means we have template injection.

Fortunately there aren't really any denied keywords in the template rendering logic itself. Depending on how you approached it, you can do some cool Python jail-escape-like maneuvers to get a shell and read the flag in the environment variable. Or you can just hit Ctrl-T and search for commonly used injections.

I used `{% raw %}{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('env').read() }}{% endraw %}`
, which was adapted from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#exploit-the-ssti-by-calling-ospopenread).

Flag: `flag{l0c4l_ip_d03s_n0t_equ4t3_t0_s4f3_ip}`


See you at next meeting.

~ josh

-----

Fun fact, someone hit a MASSIVE devious lick and yoinked the remainders of my candy before last meeting... Like 100 pieces... :(

So probably Hi-Chews or something next meeting :D
