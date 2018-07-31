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

That's it. There are no more dependencies you have to install separately.

## Installing and Configure Nextcloud

To install the snap package:

```bash
sudo snap install nextcloud
```

To finish the install - which includes setting up the admin user and adding a trusted domain - there are two options:

1) Visit your hostname in a webbrowser and enter an admin username and password. Once complete you should be able to start adding normal users and apps right away. Among other changes, the `trusted_domains` array in the config file will be updated to include the visited hostname. The config file can be found at `/var/snap/nextcloud/current/nextcloud/config/config.php`.

2) Run the `nextcloud.manual-install` command and add the hostname to a `trusted_domains` array with the `nextcloud.occ` command.

```bash
export ADMIN_NAME=admin
 export ADMIN_PASS=super-secret #leading space to not save in history
export EXTERNAL_HOST=mydomain.tld
sudo snap run nextcloud.manual-install $ADMIN_NAME $ADMIN_PASS && \
    sudo snap run nextcloud.occ config:system:set trusted_domains 1 --value=$EXTERNAL_HOST
```

`sudo snap run nextcloud.occ` gives you access to the majority of the tools required for maintaining the Nextcloud app via the command line. I'll describe most of the others in the coming sections.

Two commonly changed config values which must be set a different way are the PHP memory limit (default 128M) and the cronjob interval (default 15 min). These can be updated, for example, with.

```bash
sudo snap set nextcloud php.memory-limit=512M
sudo snap set nextcloud nextcloud.cron-interval=10m
```

Using `-1` as the value will disable either.