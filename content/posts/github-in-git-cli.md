+++
title = "GitHub from the CLI with Hub"
date = "2017-08-04T07:41:00-05:00"
tags = ["Git", "GitHub"]
slug = "github-from-cli-hub"
summary = """
The obvious go-to for GitHub is the user-friendly web interface. A command-line alternate exists however for those included called hub. This post provides instructions on how to use for making and merging a pull request.
"""
+++

## Background

For fans of the command line who are also regular users of GitHub
there's a great option for integrating Git & GitHub commands into the
same CLI workflow.

It's called [hub](https://github.com/github/hub) and it's maintained by
GitHub. While the README is okay, there a few missing details and I
couldn't find any other tutorials on getting started. So I decided to
write one!

This tutorial covers:

1. Installation
2. Configuration of auto-completion and `git` integration
3. Authentication and remote repo creation
4. More thorough usage example with GitHub Flow

Only continue if you are already familiar with Git as well as
browser-based usage of GitHub. To get up-to-speed see GitHub's own
[on-demand training courses](https://services.github.com/on-demand/)

## Install

As per the [README](https://github.com/github/hub), you can install on
Mac, Windows and Fedora with the package managers Homebrew, Chocolatey
and DNF respectively.

I'll provide instructions specific for Ubuntu (tested on 16.04 and
17.10) but you should be able to follow along with modifications for
most OSes.

First download the latest of either version 2.3.0 or the [latest stable binary for Linux](https://github.com/github/hub/releases/latest). In
this tutorial, I'll be using version 2.3.0 for some newer features
(assigning issues to a user). At this moment it's not stable (v2.2.9 is)
but should be soon.

To install we'll untar and run the `install` script, which creates the
program in `/usr/local/bin` and a manpage.

```shell
tar zxvf hub-linux-ARCH-X.Y.Z.tgz
sudo hub-linux-ARCH-X.Y.Z/install
```

You can confirm the install (your versions will likely be different):

```shell
$ hub version
git version 2.11.0
hub version 2.3.0-pre10
```

## Integrate with git command and enable auto-completion

Both these parts are optional, as we can now use all of `hub`'s
functionality. But two options to consider for a better experience are:

* aliasing it with `git`
* enable auto-completion for a better experience

For auto-completion, first we need to put the `etc/hub.bash_completion`
file included in the release somewhere permanent.

```shell
sudo cp etc/hub.bash_completion.sh /usr/local/etc
```

Now, for Ubuntu, we'll add the following lines to `~/.bashrc`.

```bash
# Alias hub with git
eval "$(hub alias -s)"

# Enable hub auto-completion
if [ -f /usr/local/etc/hub.bash_completion ]; then
    . /usr/local/etc/hub.bash_completion
fi
```

To see the effects, open a new terminal or run `source ~/.bashrc`, then
confirm success with:

```shell
$ git version
git version 2.11.0
hub version 2.3.0-pre10
```

Also hit tab after `git` and you should see auto-completion display a
list of available commands.

## Authenticate and Create Repos

To use hub we need to authenticate with GitHub. This is done by
providing your login credentials.

`hub` will store your username and an oauth token in a `~/.config/hub`.
You can delete the token anytime in https://github.com/settings/tokens
to revoke access.

### Logging in and Brand New Repository

`hub` expands certain `git` commands with additional options while
creating new ones:

* `git init` is one it expands, with the `-g` flag, hub will
  initialize a repo while automatically adding a remote
* `git create` is a new command with hub, it will create the repo on
  GitHub
* `git browse` is another new command it will open GitHub in your
  default browser

Let's try these both with our throwaway "hubapp":

```shell
git init -g hubapp # you'll be asked for your credentials if you're first time
cd hubapp
echo "# My New Hub App" > README.md #optional but let's put something in there
git add .
git commit -m "Initial commit"
git create # hubapp now exists on GitHub!
git push -u origin master
git browse # will open hubapp on GitHub in your browser
```

### Existing Local Repository

A more typical scenario is where you've done some work on a new project
locally and want to create it on GitHub. Now you can run a simple
command instead of:

* Opening up your browser
* Logging into GitHub
* Creating a project on GitHub
* Adding your remote locally
* Hopefully getting it right the first time

Let's say you're in an existing app with some killer potential called
"theNextFacebook" and want to start some GitHub collaboration.

```shell
git create # remote added and repo created on GitHub under your user's namespace
git push -u origin master # that's it!
git browse # optionally, open in your browser
```

Alternatively add an ORG/NAME argument to `git create` the repo under an
organization's namespace instead (your user will need the required
permissions in the organization).

## GitHub Flow with git+hub

In this final part we'll see that we can complete [GitHub Flow](https://guides.github.com/introduction/flow/) from the CLI with
`git`+`hub` commands. Let's pretend we're working on a small project
with a few contributors using the [Shared Repository Model](https://help.github.com/articles/about-collaborative-development-models/).

1. Create and assign an issue
2. Create and work on a topic branch
3. Sync with GitHub and rebase on master for a clean history
4. Push and create pull request
5. Merge with master and close issue

### Create an issue

First we'll **create an issue** and assign ourselves:

```shell
git issue create -a mygithubusername
```

Your default text editor will open. The first line will be the title
while the rest will be the description. For example:

```
Add fetch user action

To be called by multiple components for a customized interface
```

Just save and close and the issue is created! You can list the issues
with `git issue`.

### Work on a topic branch

Next, we'll **create a branch both locally and on GitHub**. The second
part is optional at this moment, but provides visibility to peers on
what's being worked on.

```shell
git checkout -b fetch-user-action
git push -u origin !$ # note: !$ is a bash shortcut for last word of last command
```

...we develop, test and commit changes locally...

Great. Now, everything is working nicely, so let's do a **pull request**,
but let's pretend two things have happened:

* we have multiple commits in this topic branch, including typo fixes
* buddy just pushed his own changes on another feature, so our topic
  branch is behind master

We want a clean commit history when our branch is merged into master, so
how do we achieve this?

### Rebase and Sync

First let's squash our multiple commits into one. For simplicity's sake
let's say we only have two our topic branch:

```shell
$ git log --oneline
513568e Fix typo... oops
5dda11f Finished fetch user
...older commits
```

We'll run an interactive `rebase`.

```shell
git rebase -i HEAD~2
```

Git will show you this in your text editor:

```
pick 5dda11f Finished fetch user
pick 513568e Fix typo... oops

# ... A bunch of instructions
```

All we need to do is tell Git to squash the 2nd commit, then save and
close. We'll use a special type of squash called "fixup" which also
discards the commit message, but incorporates the changes into the first
commit. If we used "squash" instead we'd have an opportunity to rewrite
the entire commit message (an unncessary step in this specific
scenario).

```
pick 5dda11f Finished fetch user
fixup 513568e Fix typo... oops
```

Running `git log` again will show one commit in the place of these two,
with a new hash but the same message "Finished fetch user".

Now let's update our master branch and use rebase again. We're using
rebase here to apply our commit on top of any interim changes made to
master by our buddy. This way we can avoid the noisy and unnecessary
"merge commit".

```shell
git sync # use the new sync command to tersely update master locally
git diff master # check to see if there are may be any conflicts warranting a different approach
git rebase master # rebase your branch commit on top of master
```

If there were no unresolvable conflicts, you'll get this message:

```
First, rewinding head to replay your work on top of it...
Applying: Finished fetch user
```

If there is an unresolvable conflict, you'll need to deal with that (Git
will help you a lot here) then use `rebase --continue`. I'm not going to
go into detail on this here, so instead I'll suggest some resources:

* For more on **the why and how of rebasing** the topic branch check
  out [this article](https://nathanleclaire.com/blog/2014/09/14/dont-be-scared-of-git-rebase/).
* Some additional instructions on resolving conflicts and [rebase --continue](http://gitforteams.com/resources/rebasing.html).

### Create Pull Request

Next, let's push and create a pull request at the same time. We'll
assign ourselves, but you can also ask for "reviewers" to check the
quality of the work with the `-r` flag. As we'll likely be practising
this flow alone the first few times let's skip the reviewers part.

```shell
git push #this pushes to topic branch remotely
git pull-request -a reviewersusername
```

Similar to the issue, you will now write your PR message. **Make sure
the body includes a reference to the original issue**, so it will be
closed automatically, for example put "**Closes #11**" in the body
where NUM is substituted with number of the issue. To get the issue
number run `git issue`.

```
Add fetch user action

Closes #1
```

### Merge and Close

Now let's finally merge the topic branch and push, closing the pull
request and issue at the same time.

```shell
git checkout master
git merge fetch-user-actions
git push
```

Note, if you are merging someone else's work on a PR assigned to you,
you would run `git sync` to ensure master is up-to-date locally, then
`git merge origin/fetch-user-actions`.

If you're happy to delete the topic branch now, just run:

```shell
git branch -d fetch-user-actions
git push origin --delete !$
```

That's it! Everything's been done via the command line. Of course mix
this up with browser-based activity where it makes sense. But if you
love the command line, having both options in your tool belt is pretty
awesome.
