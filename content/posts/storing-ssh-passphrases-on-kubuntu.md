+++
title = "Storing SSH Key Passphrases on KDE Plasma"
date = "2018-11-29T18:38:00-05:00"
tags = ["Kubuntu", "KDE", "Ubuntu", "SSH"]
slug = "storing-ssh-passphrases-on-kubuntu"
summary = """
Unlike GNOME, KDE Plasma does not have a super automatic means of storing SSH key passphrases in the user's keyring. This post includes provides quick instructions on how to do so as well as as some extra explanatory details.
"""
+++

## TL;DR

Storing your SSH key passphrases in the KWallet on Kubuntu is simple, albeit not as streamlined as with the GNOME Keyring. To do so create an executable file `~/.config/autostart-scripts/ssh-add.sh` with the following content:

```bash
#!/bin/bash
ssh-add </dev/null
```

To include more keys beyond the default `~/.ssh/id_rsa` add each as an argument, e.g. `ssh-add $HOME/.ssh/key1 $HOME/.ssh/key2 </dev/null`.

Execute the file or re-login to receive a prompt to store the passphrase in KWallet.

## Explanatory Walkthrough

### Background

In my Linux desktop journey over the last few years I've switched from Kubuntu to Ubuntu GNOME 
to the GNOME-based POP!_OS and now back to Kubuntu. 

I'm really happy with the recent switch. I find the KDE Plasma desktop experience much more natural. Admittedly, this could be due to extensive past & workplace Windows use something inherit to Plasma & GNOME. Either way, I could never get into the GNOME workflow and anticipate staying with Kubuntu for awhile.

One nice feature of GNOME was the automatic manner in which SSH key passphrases are added to the [GNOME Keyring](https://en.wikipedia.org/wiki/GNOME_Keyring).
On first usage of the passphrase, it is stored.

Plasma uses the similar KWallet (or KDE Wallet Manager) however storing passphrases is not quite so seamless.

To figure it out, I followed these instructions from the [Arch Linux Wiki](https://wiki.archlinux.org/index.php/KDE_Wallet). In the rest of this post I will apply these instructions to Kubuntu with explanation.

### Prerequisites

The Arch Linux article specifies the following pre-requisits:

1. SSH agent is up and running
2. `ksshaskpass` package is installed
3. `SSH_ASKPASS` environment variable is set to `ksshaskpass`

With the default Kubuntu install (confirmed with v18.04) the first two are already satisfied while the third is not necessary.

To clarify on the third, by default the `SSH_ASKPASS` is not set, in which case the `ssh add` command uses `ssh-askpass` as a default. On Kubuntu this is linked to `ksshaskpass`. You can confirm using `update-alternatives`:

```bash
update-alternatives --list ssh-askpass
# /usr/bin/kssaskpass
```

### Autostart Script

Shell scripts created in `~/.config/autostart-scripts/` will execute at login, exactly what is needed for this use case. 

Create a file (or symlink) here, for example:

```bash
vim ~/.config/autostart-scripts/ssh-add.sh
```

Add the following content:

```bash
#!/bin/bash
ssh-add </dev/null
```

Save, close & make the file executable:

```bash
chmod +x ~/.config/autostart-scripts/ssh-add.sh
```

The simplest case above works only for the default key `~/.ssh/id_rsa`. To include all keys provide each as a parameter to the command, e.g. `ssh-add $HOME/.ssh/key $HOME/.ssh/key2 </dev/null`.
Note that the redirect `</dev/null` is used to ensure `ksshaskpass` is used (although this should only be required if executing from the terminal).

The only thing left is to save the passphrase in the KWallet. This can be done either by executing the script from the terminal or logging out and then back in. A prompt will appear. Enter the passphrase, opt to remember it, and click OK.

{{< bootstrap/figure src="/img/kwallet-ssh-add.png" caption="Saving passphrase to KWallet" extra_figcaption_class="text-center" >}}

Now the passphrase will be read from KWallet at each login and added to the SSH Agent, no passphrase re-entry required.
