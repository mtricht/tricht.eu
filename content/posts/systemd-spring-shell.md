+++ 
draft = false
date = 2019-08-03T22:11:32+02:00
title = "Startup issue with systemd and Spring shell"
slug = "systemd-spring-shell" 
+++
Are you having issues starting up a simple Spring application through systemd (or something similar)?
And are you running Spring web and Spring shell together? I was, perhaps I shouldn't, but that's for later. It was crashing on boot when running through systemd but was working just fine whenever I ran it directly from the terminal.

The only difference being this warning in the logs:

> org.jline: Unable to create a system terminal, creating a dumb terminal (enable debug logging for more information)

It turns out that systemd doesn't like interactive shells very much, which Spring shell is providing us. I don't want a interactive shell when I'm running my application through systemd, I only want the Spring web part. After some digging, I found out that you can disable 
the interactiveness of the shell by adding the following to your `application.properties`:

```ini
spring.shell.interactive.enabled=false
```

Perhaps a better solution would be to separate the two into two jars or applications, but not when prototyping.
