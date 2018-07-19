---
title: Using the Nextcloud Server Snap
date: 2018-07-20T23:11:00-05:00
tags: [snap, linux, nextcloud]
categories: [Cloud]
slug: nextcloud-with-snap
summary: |
    Install, migrate and maintain a Nextcloud Server as a snap
---

## Background

I wrote on my experiences installing and maintaining a Nextcloud instance partially as documentation and partially to entrench learning around administering a Linux app.
As [described for Nextcloud 11]({{ ref "deploy-nextcloud.md" }}) several you need to install and configure many dependencies including (typically) Apache, PHP and MariaDB/MySQL.
You also need to set up [letsencrypt](https://letsencrypt.org/) or adminster some other OpenSSL certificates.

This is a lot to do and lot to figure out. It would be great if this was all packaged into a simple to use package. Snaps enable for most of the popular Linux distros.
It's an excellent option for a single node install due to their simplicity.
Because all the dependencies and core product are packaged together in a single relocatable app it's not an option for scaling to have different services on multiple nodes.

## Installing Snap

If you're on Ubuntu 16.04+ you already have it. For other Linux distros instructions are [available here](https://docs.snapcraft.io/core/install).

That's it. There are no more dependencies you have to set up yourself.

## Installing and Configuring Nextcloud 