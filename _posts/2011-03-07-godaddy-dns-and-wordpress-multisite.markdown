---
layout: post
title: "GoDaddy DNS & Wordpress Multisite"
date: 2011-03-07T00:00:00-06:00
---
Tonight, I decided to setup a multisite configuration for my Wordpress installation (which you are reading right now). My sister is involved in marketing and advertising and wanted to start her own blog. I figured this was a good opportunity to play with the server some more.

<!-- more -->

I decided to use the Subdomain configuration and the instructions found [here](http://codex.wordpress.org/Create_A_Network). Only, initially, I didn't want to apply a blanket `*.johnrengelman.com` DNS redirect. I don't know why I didn't want to do this. It was an arbitrary decision on my part. An arbitrary decision that ended up costing me about 2 hours tonight ! :frowning:

Somehow, as I was configuring the DNS entries (using GoDaddy's DNS Manager), I messed up something. After completing the installation in Wordpress, I couldn't reach my sister's [site](http://amanda.johnrengelman.com). I could open any other sub-domain which would redirect to my homepage.

After backtracking and restoring backups and reinstalling, I still couldn't open the site. The confusing part is that `nslookup` resolved the address correctly, but `ping` couldn't reach my server. :confused: Very confusing.

It appears that some entry I placed into the DNS records propagated out and wasn't immediately updated when I entered new settings. I decided to give up for a few minutes and when I came back, everything was working. Ugh. Server work is very frustrating at times.
