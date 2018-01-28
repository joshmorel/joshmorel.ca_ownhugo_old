---
title: Update Nextcloud
date: 2017-04-14T12:38:00-05:00
moddate: 2018-01-27T05:49:00-05:00
tags: [nextcloud]
categories: [Cloud]
slug: nextcloud-update
summary: How to update Nextcloud and restore if needed
---

**Update:** Since writing this I've learned that future major version upgrades (e.g. 12->13) will be a lot simpler so most of these instructions have lost their relevance. Even so, doing [your own backup]({{< ref "#pre-update" >}}) rior to upgrading is worthwhile insurance IMHO.

## Background

After learning about a (minor?) Nextcloud controversy through the
[Linux Action Show podcast episode 460](http://www.jupiterbroadcasting.com/107471/nextclouds-can-of-worms-las-460/)
I decided to update my Nextcloud instance with the latest patch.

Essentially, Nextcloud developed a scanner to look for vulnerabilities
in internet-hosted ownCloud and Nextcloud (i.e. not intranet) instances
findable through services such as [Shodan.io](https://www.shodan.io/).
The vulnerabilities were identified based on the HTTP headers and HTML
body of the login page only.

The controversy was largely fueled by the fact that those with insecure
servers were contacted through their ISPs by Nextcloud. This,
understandably, made some people very upset. However, I feel that
insecure ownCloud/Nextcloud servers are not just a ticking time-bomb for
the users but for open source sync-n-share and the Nextcloud brand more
generally. Although the approach may not have been perfect, action on
their part was certainly warranted in my opinion. More details and
opinions can be found here:

* https://help.nextcloud.com/t/someone-scans-the-internet-for-nc-oc-instances/8992/24
* https://www.reddit.com/r/selfhosted/comments/5ybmf1/nextcloud_scanning_peoples_owncloud_and_nextcloud/

## Pre-Update

First to note, this is an update within a major release (version 11) so
the process is different than what might be required for a moving from
one major version to another.

Before continuing, you should ensure the Nextcloud database and file
directory are backed-up. Regular automated back-ups are _highly
recommended_, but are beyond the scope of this article. I'm considering
an article on this in the future.

No matter your set-up, do a temporary one-time back-up before updating.
I'll actually cover recovering from a bad update at the end of this
article based dependent on this first step. Now, these and subsequent
steps assume you followed my [deployment article]({{< ref "deploy-nextcloud.md" >}}) (are using Ubuntu, Apache
and MariaDB) and have sufficient storage space available. If not you'll
have to revise as needed but the principles are the same.

```shell
su - # become root
rsync -Aax /var/www/nextcloud/ /var/backups/nextcloud-dirbkp/ # back-up directory
mysqldump --lock-tables -h localhost -u root nextcloud > /var/backups/nextcloud-sqlbkp.bak # back-up database
```

## Update using Web Interface

The update process [described here](https://docs.nextcloud.com/server/11/admin_manual/maintenance/update.html)
is largely automated, however, a few additional steps are required. Also
note that 3rd party apps will be disabled (although no data will be
deleted). At the moment, re-enabling them manually is an acceptable
choice.

Run this script (as root) to allow the HTTP user to make changes to your
file directory.

```bash
#!/bin/bash
# Sets permissions of the Nextcloud instance for updating

ocpath='/var/www/nextcloud'
htuser='www-data'
htgroup='www-data'

chown -R ${htuser}:${htgroup} ${ocpath}
```

Then log-in to the Nextcloud web interface as admin and navigate to the
Admin page. In this case you will see that I'm not on the latest version
and can update here.

![image: Nextcloud Update Version Page](/img/nextcloud_update_version_page.png)

---

1. Click "Open updater" then "Start update".
2. When prompted for "Keep maintenance mode active?" click "No (for usage of the web based updater)".
3. Click "Go back to your Nextcloud instance to finish the update" then click "Start update"
4. When update completed, note the apps that have been disabled and click "Continue to Nextcloud"
5. Re-enable all the 3rd party apps noted in step 4

If you get a `Maintenance Mode` notice during step 3 just wait a few
moments and the page should refresh with the `Start update` options.

Finally, re-harden the server with the [following script from Nextcloud](https://docs.nextcloud.com/server/11/admin_manual/installation/installation_wizard.html#strong-perms-label).

### Harden Server

```bash
#!/bin/bash
# For hardening security on Nextcloud 11

ocpath='/var/www/nextcloud'
htuser='www-data'
htgroup='www-data'
rootuser='root'

printf "Creating possible missing Directories\n"
mkdir -p $ocpath/data
mkdir -p $ocpath/updater

printf "chmod Files and Directories\n"
find ${ocpath}/ -type f -print0 | xargs -0 chmod 0640
find ${ocpath}/ -type d -print0 | xargs -0 chmod 0750

printf "chown Directories\n"
chown -R ${rootuser}:${htgroup} ${ocpath}/
chown -R ${htuser}:${htgroup} ${ocpath}/apps/
chown -R ${htuser}:${htgroup} ${ocpath}/config/
chown -R ${htuser}:${htgroup} ${ocpath}/data/
chown -R ${htuser}:${htgroup} ${ocpath}/themes/
chown -R ${htuser}:${htgroup} ${ocpath}/updater/

chmod +x ${ocpath}/occ

printf "chmod/chown .htaccess\n"
if [ -f ${ocpath}/.htaccess ]
 then
   chmod 0644 ${ocpath}/.htaccess
   chown ${rootuser}:${htgroup} ${ocpath}/.htaccess
fi
if [ -f ${ocpath}/data/.htaccess ]
 then
   chmod 0644 ${ocpath}/data/.htaccess
   chown ${rootuser}:${htgroup} ${ocpath}/data/.htaccess
fi
```

You should be good to continue use with a patched Nextcloud!

## Recover from Bad Update

If the update went bad for any reason it's easy to recover given that
you've back-up both the database and file directory as described above.

First clean and restore the database:

```shell
mysql -h localhost -u root -e "DROP DATABASE nextcloud"
mysql -h localhost -u root -e "CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci"
mysql -h localhost -u root nextcloud < /var/backups/nextcloud-sqlbkp.bak
```

Then restore the Nextcloud directory. This will restore the entire
directory to its state prior to the update. I've included the `--delete`
flag here which deletes extraneous files in the destination not in the
source (back-up). This is different from what is suggested in the
official Nextcloud documentation but I feel this is necessary if you
want the restored version to behave as expected. If any readers disagree
please provide feedback!

```shell
rsync -Aax --delete /var/backups/nextcloud-dirbkp/ /var/www/nextcloud/
```

Finally, [Harden Server]({{< ref "deploy-nextcloud.md#install-nextcloud" >}}) as when installing to ensure secure directories.

If you want to restore to a previous version after any period of use on
the latest version (i.e. downgrading) then that is a FAR trickier
situation and simply removing the `--delete` flag would not be
sufficient. You're likely better off fixing whatever is causing issues
rather than trying to downgrade while preserving the latest data. There
certainly are ways but the approach would be largely
situation-dependent.

### Key Recommendation

Practice! Do this in a "QA" environment a few times before updating
production.

## Next steps

I'm considering one of two things for a future Nextcloud-related
article:

1. Nextcloud automated deployment with [Ansible](https://www.ansible.com/)
2. Making regular, automated back-ups
