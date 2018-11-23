---
title: Migrate Static Site from GitHub Pages to S3 and CloudFlare
date: 2017-05-17T19:44:00-05:00
tags: [Blogging, Pelican, Jekyll, S3, Cloudflare]
slug: migrate-github-page-s3-cloudflare
summary: |
    Step by step instructions to migrate static site from github page to
    s3 and cloudflare
---

## Background

Back in November 2016 I wrote my first how-to article on creating a
[GitHub Personal Page]({{< ref "create-github-page.md" >}}).

My local set-up included a:

* Git repo with content and configuration files
* Git submodule with generated output to host on GitHub
* Git submodule with pelican-plugins

After reading through an article on static site hosting with [S3 and CloudFlare](https://wsvincent.com/static-site-hosting-with-s3-and-cloudflare/)
for "pennies a month" (caution: not yet verified!) I decided this was
something I wanted to do.

My reasons include:

* Git submodule repo for constantly regenerated output was an ugly and awkward solution
* Can use custom personalized domain combined with SSL - more professional
* Increase skills with popular services - S3 and CloudFlare

In this article I'll describe how to use:

1. [S3]({{< ref "#setup-amazon-s3" >}}) to host your site
2. [Pelican]({{< ref "#publishing-to-s3-with-s3cmd-and-pelican">}}) and s3cmd to publish
3. [CloudFlare]({{< ref "#leveraging-cloudflare-content-delivery">}}) to serve content over SSL
4. [Jekyll]() to redirect from GitHub to your new site

This article is a "buffet" of content and not all might be pertinent
to your needs. A leaner how-to without the redirect topic is [available here](https://kawashi.me/how-to-host-your-blog-using-pelican-amazon-s3-and-cloudflare.html).

## Setup Amazon S3

First create an [Amazon S3 account](https://aws.amazon.com/s3/). If you
are new to AWS then you can get 12-months of 5GB storage & 20k get
requests per month free. This should be more than enough for a personal
site. After that there [is a cost](https://aws.amazon.com/s3/pricing/)
but it's reasonable.

Like Will Vincent, the author of the original article, I created two
buckets with with just my domain `joshmorel.ca` and another with the
`www` subdomain which redirects to the first.

### Static Site Bucket

To achieve a similar affect, click the `Create Bucket` (1), enter your
domain for `Bucket name` (2), and click `Next` through the rest of
the screens.

![image: Create S3 Bucket](/img/s3-create-bucket.png)

---`

Click on the bucket, then navigate to `Permissions` (1) > `Bucket Policy` (2) and paste the following text into the `Bucket policy editor`, substituting with your actual domain (3). This essentially allows any browser to GET any object in the bucket (which is what we want). Be sure to click `Save` (4).

![image: Set S3 Bucket Policy](/img/s3-bucket-policy.png)

---

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AddPerm",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::yourdomain.com/*"
    }
  ]
}
```

---

To enable static hosting for this bucket, go to **Properties** (1),
click **Static website hosting** (2), select the **Use this bucket to
host a website** option (3), enter `index.html` and `404.html` for
**index document** (4) and **Error document** (5) respectively, then
click **Save** (5). Note that you will need to use a 404.html template
to serve a custom 404 page, which I won't cover in this article.

![image: Enable S3 Static site hosting](%7Bfilename%7D/images/s3-static-hosting.png)

### Redirect Bucket

Optionally you can create a `www.yourdomain.com` bucket which simply
redirects to `yourdomain.com`.

To do so create a new bucket similarly to above with the `www` subdomain
as the bucket name. Then open the bucket, navigate to `Properties`,
click `Static website hosting`, select the `Redirect requests`
option and enter your domain for the `Target bucket or domain`.

## Publishing to S3 with s3cmd and Pelican

Although you can upload files manually to S3 this is a horribly
inefficient option. Instead, I recommend using the
[s3cmd](http://s3tools.org/s3cmd) tool (also written in Python) combined
with the Pelican
[Makefile](http://docs.getpelican.com/en/stable/publish.html#make).
Using `Makefile` on Windows may be tricky but there should be tips elsewhere on the web.

### Enable s3cmd Management

To use s3cmd you must both install s3cmd and set up AWS Identity and
Access Management (IAM). To install s3cmd, on Ubuntu 16.04 I used
`sudo apt install s3cmd`. For others, you can check your package manager
or visit [the Download page for other options](http://s3tools.org/download).

You also need credentials for remote management, created through IAM.
Log on to the AWS console navigate to the IAM service. You can also add
required credentials to an existing user, but assuming you don't have
one, click `Users` (1) then `Add user` (2).

![image: Create AWS user](/img/s3-iam-create-user.png)

---

Enter a meaningful name (1) and check off `Programmatic access` (2)
then click `Next: Permissions` (3).

![image: Set AWS user details](/img/s3-iam-set-user-details.png)

---

Click `Attach existing policies directly` (1), filter on "S3 (2), then
check `AmazonS3FullAccess` (3), then scroll down and click `Next: Review`. On the final screen click `Create user`.

![image: Set AWS user S3 permissions](/img/s3-iam-set-user-permissions.png)

---

After successful creation, download the .csv file or copy the `Access key ID` and `Secret access key`. Either way, it is important to keep
these secret. These credentials can allow any agent to create and use
practically unlimited S3 buckets under your account as well as
compromise existing buckets.

Back in the terminal run `s3cmd --configure` and provide the necessary
details for each prompt. For more information about the different
options visit the [s3cmd how-to](http://s3tools.org/s3cmd-howto). For
region codes to use when prompted for `Default Region` see the [AWS S3 docs](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region).
These steps save your configuration to `~/.s3cfg`.

### Publish to S3 with Pelican Makefile

Previously I was generating the site with `pelican` commands, reviewing
the output with `python -m SimpleHTTPServer` and publishing with
`git push`. Now I will be using `Makefile` commands to do everything. If
you don't already have the file run `pelican-quickstart` providing
similar answers to the S3-related prompts (lines beginning `>`):

```shell
pelican-quickstart
> Do you want to upload your website using S3? (y/N) y
> What is the name of your S3 bucket? [my_s3_bucket] yourdomain.com
mv Makefile develop_server.sh path/to/your/siterepo
```

A few extra notes:

* The `develop_server.sh` script makes serving locally easier,
  executed with the `make devserver` command
* If your `content` and `output` sub-directories are not named
  `content` or `output`, respectively, then you will need to edit
  these files

To publish your content to S3:

```shell
cd path/to/your/siterepo
make publish
make s3_upload
```

You should now be able to see your see your site index file at your
bucket endpoint, for example:
http://s3.ca-central-1.amazonaws.com/yourdomain.com/index.html. But
this is less than ideal. We want to be able to use our own domain name
plus SSL. To achieve this wa can use a free tier from CloudFlare.

## Leveraging CloudFlare Content Delivery

### Create CloudFlare Site

CloudFlare is a content delivery network (among other things) with a
free-tier that should be sufficient for a personal site. This will
enable us to use our own domain and SSL while taking advantage of some
modest gains in performance and protection against DDoS (compared to
self-hosting). Create an account through
[CloudFlare's%20sign-up%20page](https://www.cloudflare.com/a/sign-up).

Using your own registered domain, click `+ Add Site` (1), enter the
domain (2) and click `Begin Scan` (3). If the domain is registered,
you should see the domain listed.

![image: Add Site to CloudFlare](/img/cloudflare-add-site.png)

---

Delete any other records (for example, the `A` record) then create a
`CNAME` record for each bucket. If you're following along, that's one
for `yourdomain.com` (you can enter `@`) with a value of
`yourdomain.com.s3-ca-central-1.amazonaws.com` (1) and one for `www`
with a value of `www.yourdomain.com.s3-ca-central-1.amazonaws.com` (2).
Be sure to use your domain/bucket as well as the region your AWS service
is located in instead of `ca-central-1`. You also must exclude the
`http` prefix (3) or any trailing slash (4). Then click `Continue`.

![image: Setting CloudFlare DNS Records](/img/cloudflare-set-dns-records.png)

---

Select the `Free Plan` then click `Continue` to arrive at the DNS
nameserver section. Note the DNS nameservers then click `Continue`
again. The site is created, but we must update our domain to use these
nameservers.

### Update Domain Nameservers

How to update the nameservers for your domain will depend on the
registrar but at a high-level it should be similar. These are:

1. For the domain of interest, delete existing DNZ zones
2. Navigate to the manage nameservers page for that domain
3. Update nameserver 1 and 2 with values noted earlier during
   CloudFlare site creation
4. Wait

We need to wait for both the domain registrar and CloudFlare updates to
synchronize. This can be quick but you are warned it can take over 24
hours. For me, actually, it took overnight so patience is necessary for
this part.

### HTTP Redirecting

Once you have confirmed your site is serving your blog
properly, there is one more step to take. We want to redirect HTTP
traffic to HTTPS.

First click on `Crypto` (1) then change `SSL` (2) to `Flexible`. We
want to _accept_ both HTTP and HTTPS traffic, we're just going to
_redirect_ HTTP to HTTPS.

![image: Change CloudFlare SSL to Flexible](/img/cloudflare-flexible-ssl.png)

---

Next click on `Page Rules` (1), then `Create Page Rule` (2). Enter a
wildcard value for URL matching (3), select `Always Use HTTPS` (4)
from the drop-down then `Save and Deploy` (5). Now when you enter
`http://yourdomain.com` you should be redirected.

![image: Set CloudFlare Page Rule to use HTTPS](/img/cloudflare-always-use-https.png)

---

### Redirect from GitHub Pages with Jekyll

If you were previously serving your site from "username.github.io" and
want to redirect visits to this page, you still need to create a Jekyll
GitHub Pages site, but it's very simple.

[Jekyll was always](https://jekyllrb.com/) the more elegant option for
serving a site from GitHub with the built-in support, but I switched to
Pelican because I wanted to use something written in a language I
understood so I could troubleshoot or customize. This is a revisit to
Jekyll, combined with
[jekyll-redirect-from](https://github.com/jekyll/jekyll-redirect-from),
but it's for the single purpose of HTTP redirection.

First [delete the existing GitHub repo](https://help.github.com/articles/deleting-a-repository/) to keep
things clean, then recreate it as an empty repository.

Instead of detailing all the different files you need to create, I'm
providing a "starter" from my GitHub repo to download:

```shell
wget https://github.com/joshmorel/joshmorel.github.io/archive/starter.tar.gz -O starter.tar.gz
tar xvzf starter.tar.gz
mv joshmorel.github.io-starter username.github.io
cd username.github.io
```

Download and install
[Ruby](https://www.ruby-lang.org/en/documentation/installation/) if you
don't already have it, edit the `index.md` (substituting your actual
domain), then run these commands to install dependencies and build and
serve to confirm it's working:

```shell
gem install bundle
bundle install # install dependencies from Gemfile
jekyll build
bundle exec jekyll serve
```

Once satisfied you can create the git repo, stage and commit changees,
then push to GitHub (substituting with actual username):

```shell
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:username/username.github.io
git push -u origin master
```

One **very important note**, wildcard re-directs are not possible so if
you want to redirect specific articles or apges you will need to create
one for each, similar to `index.md`. I _might_ write a script to help
with this, but for now, please feel free to contact me if you have any
question!

That's it, you should be good to use your newly personalized and
optimized site!
