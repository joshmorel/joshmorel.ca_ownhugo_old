+++
title = "Node Virtual Environments with nodenv"
date = "2017-03-04T18:38:00-05:00"
tags = ["Node.js", "nodenv"]
slug = "node-virtual-environments-with-nodenv"
summary = """ Using nodenv for Node.js virtual environments in Ubuntu
I use nodenv for Node.js version and virtual environment management. In this post I briefly compare considered alternates and provide instructions on how to set nodenv up.
"""
+++

## Background

I'm becoming a huge fan of Node.js but deciding on the best option for
working with versions and virtual environments can be a bit confusing.
Much like everything else in the Node.js ecosystem, there are more than
enough options to choose from. To cement my own understanding and
perhaps provide some help to others I'll outline how to use
[nodenv](https://github.com/nodenv/nodenv) for managing virtual
environments for Ubuntu. This should work equally for most Linux distros
and Mac should be similar. Windows would require an alternate approach.

I'll cover deployment considerations in another article.

### Alternates

I found nodenv to be fast, usable and very well documented.
Additionally, it is forked from rbenv so provides a familiar interface
for those also working in Ruby.

But before continuing, I'll recognize three alternates which I
considered and also tried.

* [nvm](https://github.com/creationix/nvm): This is the one you will
  see the most articles about. I did work for me in the short time I
  used it but I do see many open issues and generally I liked nodenv
  better.
* [nave](https://github.com/isaacs/nave): Very similar to nvm and it
  also seems good. However it wasn't as user-friendly as either nvm or
  nodenv.
* [n](https://github.com/tj/nv): This is the only one I would
  explicitly not recommend at my time of writing this. It doesn't have
  the functionality to easily manage virtual environments for a
  variety of projects (e.g. it only keeps one set of node_modules).
  It could be an option for managing node versions at the system-level
  only but I'd rather use an apt repository for that (see later).

I'll also note that none of these options work on Windows - only OS X &
Linux - so if you're working on Windows you may consider
[nvm-windows](https://github.com/coreybutler/nvm-windows) or
[nodist](https://github.com/marcelklehr/nodist). `nodenv` _might_ work
with MinGW/Cygwin.

### Ubuntu 16.04 - Default apt-based install

**Important**: The next two commands are for demonstration only. You do not need to do them.

To install Node.js from the main Xenial repository at the system level
you can either run apt install nodejs or apt install npm.

npm is the package manager for node and is required to work in the
Node.js ecosystem more generally. However, we'll get the later version
of npm with nodenv later so let's just install Node.js to keep things
tidy.

```shell
sudo apt install nodejs
```

You can confirm that Node.js was installed:

```shell
$ nodejs -v
v4.2.6
```

So is v4.2.6 a recent version? Not really. You can visit the
[<https://github.com/nodejs/LTS/>](Node.js%20Long-term%20Support%20Working%20Group)
to confirm.

It is active LTS but ending in 2017-03-31 so maybe we want something a
touch more current? As of this writing v6.10.0 is the latest LTS
version.

## Installing nodenv & node-build plugin

There are a few steps to the nodenv install. However, it's well worth it
given the end result. I actually like how nodenv carries on the Unix &
Node.js philosophies of program doing one thing well. For this reason,
there are a few plugins required to get the functionality I desire.

First install & activate nodenv by cloning the git repo, appending the
required lines to your .profile.

If you are using CentOS/REHL or Mac then you'll need to edit your
.bash_profile instead.

```shell
git clone https://github.com/nodenv/nodenv.git ~/.nodenv
echo 'export PATH="$HOME/.nodenv/bin:$PATH"' >> ~/.profile
echo 'eval "$(nodenv init -)"' >> ~/.profile
```

Log out then log back in to confirm the install:

```shell
$ type nodenv
nodenv is a function
nodenv ()
...
```

As mentioned earlier nodenv is modular with plugins performing some of the essentials functions, including the actual building of node versions. We can install
[node-build](https://github.com/nodenv/node-build#readme) and then list
available versions.

```shell
$ git clone https://github.com/nodenv/node-build.git $(nodenv root)/plugins/node-build
$ nodenv install -l
0.1.14
0.1.15
...
```

### Installing Node Versions & Packages

Now that we have nodenv and the node-build plugin, we can install
multiple Node.js versions. Let's install the latest LTS version (at time
of this writing) and then activate it globally (for the logged in user).

```shell
nodenv install 6.8.0
nodenv global 6.8.0
```

You can confirm node & npm are installed:

```shell
$ npm version
{ npm: '3.10.10',
  ares: '1.10.1-DEV',
  http_parser: '2.7.0',
  icu: '58.2',
  modules: '48',
  node: '6.10.0',
  openssl: '1.0.2k',
  uv: '1.9.1',
  v8: '5.1.281.93',
  zlib: '1.2.8' }
```

To install a global node module we do so as per usual, but we need to
rehash to make it available from the command-line.

```shell
$ npm install -g mocha
$ mocha -V #without rehash
mocha: command not found
$ nodenv rehash
$ mocha -V
3.2.0
```

We can install the nodenv-package-rehash plugin to enable auotmatic
rehashing.

```shell
$ git clone https://github.com/nodenv/nodenv-package-rehash "$(nodenv root)"/plugins/nodenv-package-rehash.git
$ nodenv package-hooks install --all
$ npm install -g nodemon
$ nodemon -v
1.11.0
```

### Installing Local Node Versions per Project

To start to see the power in nodenv let's install a project locally for
a specific project.

```shell
$ nodenv install 4.8.0
$ mkdir path/to/projects/hello-nodenv
$ cd path/to/projects/hello-nodenv
$ node -v #displaying global
v6.8.0
$ nodenv local 4.8.0
$ node -v
v4.8.0
```

This will write `4.8.0` to a .node-version file. When you cd into this
directory nodenv will adjust the version shim (for details on how this
works see the `README`).

Now all our npm scripts will work with the local version instead of the
global version.

We can see how this works by:

1. Create a new project with `npm init`
2. In package.json add `"start": "node index.js"` within `scripts`
3. Then create a simple program and run it

```shell
$ echo 'console.log("Hello from " + process.version);' > index.js
$ npm start
...
Hello from v4.8.0
```

## Next steps - nodenv for deployment to production

How you install Node.js for deployment to production will vary based on
your application's overall architecture. If you deploy to Heroku, for
example, the Node.js version and all dependencies will be installed
automatically based on what is in your package.json file.

In other scenarios (deployment to cloud provider instance, for example),
you can leverage nodenv to facilitate automatic installation of the
correct Node.js version.

In the next article on this topic I will write on a simple production
deployment using nodenv plus process management using
[pm2](http://pm2.keymetrics.io/).
