+++ 
draft = false
date = 2019-07-21T16:48:45+02:00
title = "Software I self-host on my personal VPS"
slug = "software-i-self-host-on-my-personal-vps" 
+++

I try to avoid cloud services because I want to be in control of my data. This is why I self-host software on my personal VPS whenever there is a viable self-hosted alternative. This VPS is nothing impressive but does its job well. It runs the latest CentOS on a VPS with 1 vCPU, 2GB of RAM, a 50GB SSD and 2TB of bandwidth costing me about $15 per month (including backups). After hopping between several VPS hosts, I stuck with [DigitalOcean](https://m.do.co/c/eb6358832805) (this is a referral link, giving you $50 credit when signing up!).

So, what do I run?

## Caddy as a web server
Before I start listing software, I need to mention [Caddy](https://caddyserver.com/).
Caddy is a HTTP/2 web server with automatic SSL certificates using [Let's Encrypt](https://letsencrypt.org/).
With a few configuration lines you can get up and running in no time.

I run Caddy in a docker container that terminates SSL and proxies requests to other docker containers.

## Bitwarden for passwords
[Bitwarden](https://bitwarden.com/) is a password manager like no other. It's open source.  Bitwarden provides an auto-fill service on all major browsers, mobile vendors and more making it easy to generate new passwords, save them and use them on all your devices. Bitwarden also includes tools such as exposed/reused/weak passwords reports and a data breach report, giving you insight in how secure your passwords are.

<div style="text-align:center"><img src="/images/bitwarden-screenshot.png" /></div>

There is one downside to Bitwarden, which is that it **requires** a Microsoft SQL server to be running. This means that you will need at least 2GB of RAM. Aside from that the installation starts **10** docker containers, which was a no go for my small VPS server. 

Thankfully, a developer made [bitwarden_rs](https://github.com/dani-garcia/bitwarden_rs). An unofficial rewrite in Rust that is compatible with Bitwarden's API but with a smaller footprint, requiring no MSSQL server. Installing bitwarden_rs is straightforward and requires only 1 docker container.

Migrating to Bitwarden is easy. Importing your passwords from other sources is just one click. See a list [here](https://help.bitwarden.com/article/import-data/) for supported sources.

## Nextcloud for file sharing and more
I had been using a lot of Google's services for a major part of my life. I used to use:

- Google Calendar
- Google Drive
- Google Contacts
- Google Keep

daily until I found out about [Nextcloud](https://nextcloud.com/). Nextcloud provides the same functionality while you remain in control of your data. A standard Nextcloud installation gives you files (Google Drive equivalent), contacts (Google Contacts equivalent) and calendar (Google Calendar equivalent). It's written in PHP and requires a MySQL database meaning you will need two docker containers.

Aside from the three functionalities mentioned above, there is an [App Store](https://apps.nextcloud.com/) with loads of apps that add more functionality. You can customize your Nextcloud instance however you'd like. In my case I really like the [Notes](https://apps.nextcloud.com/apps/notes) (Google Keep equivalent) and [Bookmarks](https://apps.nextcloud.com/apps/bookmarks) apps, which both have accompanying mobile apps.

Already having switched from Google's search engine to DuckDuckGo, I only have Gmail left to replace, which I plan to do later this year.

![Nextcloud screenshot](/images/nextcloud-screenshot.png)

## The Lounge for IRC
[The Lounge](https://thelounge.chat/) is a self-hosted web IRC client, which is always connected. If you're still active on IRC and like things to be accessible through the web, this is the IRC client for you. You won't be needing an IRC network bouncer like ZNC, as you will stay connected. It's easy to install with many options, docker being the easiest to setup.

![The Lounge screenshot](/images/thelounge-screenshot.png)

## Others
Other pieces of software that I self-host that are worth mentioning but are rather small:

- [Hastebin](https://github.com/seejohnrun/haste-server) - pastebin-like sharing of text files. Written in NodeJS, runs in docker.
- [Teamspeak](https://www.teamspeak.com/en/) - VOIP for online gaming. The only that is proprietary in this list.
- [Fathom](https://github.com/usefathom/fathom) - Web analytics.
- [CyberChef](https://github.com/gchq/CyberChef) - a web app for encryption, encoding, compression and data analysis 
- [Jellyfin](https://jellyfin.org/) - media system, where nextcloud files are mounted as readonly

## Wishlist
Having listed of the software I self-host, there are still a few on my wishlist to try out:

- [Gitea](https://gitea.io/en-us/) - "Git with a cup of tea. A painless self-hosted Git service." which looks an awful lot like GitHub.
- [Drone](https://drone.io/) - "Continuous delivery · built on docker"

If you're looking for more software to self-host, checkout the [awesome-selfhosted](https://github.com/Kickball/awesome-selfhosted) page on GitHub.
