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

The snap command `nextcloud.occ` gives you access to the majority of the tools required for maintaining the Nextcloud app via the command line. I'll describe most of the others in the coming sections.

Two commonly changed config values which must be set a different way are the PHP memory limit (default 128M) and the cronjob interval (default 15 min). These can be updated, for example, with.

```bash
sudo snap set nextcloud php.memory-limit=512M
sudo snap set nextcloud nextcloud.cron-interval=10m
```

Using `-1` as the value will disable either.

## Installing SSL Certificates

One really awesome feature of the Nextcloud snap is thje tooling to install and update a [letsencrypt](https://letsencrypt.org/) SSL certificate.

Besides letsencrypt you can use the `nextcloud-enable-https` command to install and configure a self-signed or custom certificate. For letsencrypt or a custom certificate, the hostname must be externally available and owned by you. You will need to purchase the name from a domain registrar and add a DNS [A record](https://support.dnsimple.com/articles/a-record/) mapping to your host's IP address (this service - DNS hosting - is typically offerred free by the domain registrar).

If you are migrating an existing instance you will want the hostname to be the same. 
Personally, I'm hosting on a Digital Ocean droplet and will use the [floating IP functionality](https://www.digitalocean.com/docs/networking/floating-ips/overview/) to seamlessly migrate from the old host to the new host. You can migrate to the same host or a new host w

and after doing some initial testing I've decided the simplest approach is to just migrate on the same instance (taking a snapshot first of course). Migrating to a different host may make more sense, in that case you'll want to update the to a new IP.

Once DNS is set up, just run this command respond to the three prompts:

```bash
sudo snap run nextcloud.enable-https lets-encrypt
```

That's it! There is even a little service included in the Nextcloud snap that will check daily to see if the certificate should be renewed and if so renew it automatically.

## Migrating to snap from classic LAMP stack

If you have an existing instance which is installed and maintained as a traditional LAMP (Linux, Apache, MySQL/MariaDB, PHP) migrating the data to a a snap instance is not that hard. I'll refer to this as the **classic** version in contrast to the **snap** version from here on out.

### Planning

Because I am using Digital Ocean for hosting my Nextcloud instance I decided to use the [floating IP](https://www.digitalocean.com/docs/networking/floating-ips/overview/) functionality to facilitate a migration to a brand new host. The benefit of the floating IP is that I can perform a more seamless switch from old host to new as I don't need to change my DNS records to retain the same hostname. Another option is to migrate in place on the same host - just be sure to properly backup or take a snapshot before continuing.

You should make sure both the snap destination and classic source are on exactly the same version.
Migrating to a newer minor version might work but to decrease the likelihood of running into any kind of issue it's best to keep them the same.
Right now the snap stable channel lags a bit behind the classic stable channel. You may need to upgrade the just installed snap package to the version in the beta channel. 

To check the current and available versions run:

```bash
snap info nextcloud
```

To upgrade to the version in the beta channel if necessary run:

```bash
sudo snap refresh --beta nextcloud
```

### Data Migration

Before continuing with the actual migration put both instances in maintenance mode.

On the classic source:

```bash
sudo -u www-data php occ maintenance:mode --on
```

On the snap destination:

```bash
sudo snap run nextcloud.occ maintenance:mode --on
```

Next migrate the data - both the user files stored within the `datadirectory` and the MySQL database.

From the source, we'll require sudo to access the data but we should be sending data to a non-root user on the destination to follow security best practices. First exchange the required keys and then run this rsync command:

```bash
# exclude updater related data/logs
sudo rsync -av --exclude=updater* -e ssh /var/www/nextcloud/data user@newhost.com:~/
```

On the new host, it'll be easiest to just to overwrite the entire existing data directory. For snap packages, the directory should be owned by root instead of the `www-data` user.

```bash
export DATA_DIR=/var/snap/nextcloud/common/nextcloud/data
sudo rm -r "$DATA_DIR"
sudo mv ~/data "$DATA_DIR"
sudo chown -R root:root "$DATA_DIR"
```

The `appdata_*` directory is suffixed with the Nextcloud `instanceid`, we'll update the destination to match. 

```bash
export INSTANCEID=oc003ous0gb1
sudo snap run nextcloud.occ config:system:set instanceid --value="$INSTANCEID"
```

Next we dump the MySQL/MariaDB database and restore it. MySQL is included in the snap and it's the only option - but a MariaDB source will definitely work.

```bash
sudo mysqldump --lock-tables -h localhost -u root -p secret nextcloud > nextcloud-sql.bak
scp nextcloud-sql.bak user@newhost.com:~/
```

Prior to restoring from the dump script we want to drop all existing tables. We don't want to drop and recreate the database as settings & permissions have been specifically setup during the snap install and we don't want to try to recreate that.

```bash
sudo snap run nextcloud.mysql-client -N -e "SELECT concat('DROP TABLE IF EXISTS ', table_name, ';') FROM information_schema.tables WHERE table_schema = 'nextcloud';"
sudo snap run nextcloud.mysql-client nextcloud < droptables.sql
sudo snap run nextcloud.mysql-client nextcloud < nextcloud-sql.bak
```

All the data is now migrated. A few final steps are required before it's ready to use. 

### Final Steps

First, depending on you're migration path, you'll want to:

1) If you're using a floating IP, simply update the floating IP via your VPS to the new host
2) If you're migrating to a new host without a floating IP, update your DNS record to the new host's IP
3) If you're migrating on the same host, you would have already needed to stop the non-snap `apache2`/`httpd` services for the Nextcloud snap to run but you'll also want to disable/uninstall these non-snap applications as they are no longer needed (Apache, MariaDB/MySQL, certbot, PHP modules)

Second, install the SSL certificate with the snap command as [described above]({{< ref "#installing-ssl-certificates" >}}).

Third, turn off maintenance mode:

```bash
sudo snap run nextcloud.occ maintenance:mode --off
```

Finally, make sure all the enabled apps in the snap match those enabled in the classic source. 

This actually turned out to be fairly simple. Log into the Web UI, then click the top-right avatar icon followed by the "Apps" item.
You'll have to browse to each category (e.g. "Office & text") but when you do you'll see those apps you copied your data over for listed first with "This app has an update available".

![Nextcloud calendar app update](/img/nextcloud-calendar-update.png)

Click "Update to x" button for each and you're done the migration! 

To be sure you've enabled all the extra apps, you can compare the app lists. In the classic source:

```bash
sudo -u www-data php occ app:list
```

On the snap destination:

```bash
sudo snap run nextcloud.occ app:list
```

## Backing Up

## Upgrading

