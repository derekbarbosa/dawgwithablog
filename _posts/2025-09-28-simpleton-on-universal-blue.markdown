---
title:  A simpleton's look on Universal Blue
date:   2025-09-28 2:45:54
categories: [universal-blue]
tags: [containers, distrobox, universal-blue, atomic, bootc, rambling]
author: Derek
---

Well, I recently converted to the church of atomic/immutable distros. What can I
say? They're actually pretty neat.

One of the central themes (which I've dabbled with through the use of `toolbox`
containers in the past) is that you're supposed to spin up a `distrobox` (same
but different) container for everything, and have that be your working
environment, while symlinking the hosts' container runtime to that container,
allowing you to run containers "within" your containerized build-enviornment.

> *Yo Dawg, I heard you like containers, so we put containers in your
> containers!*

This is pretty cool.

However, there's a bit of a mental-model shift here. Not entirely neccessary,
but if we're putting the effort of moving into an immutable distro -- we should
really strive for best practices.

Traditionally, on mutable systems (or just standard distros, if you're trying to
avoid unneccessary jargon) you would just bring in your config files for
everything --  your editor, mail client, etc etc. From there, you would just
have a bunch of different git repos cloned in the directories of your choice.

For example, let's say this is a system that you recently provisioned. You would probably
globally install some packages using a package manager.

For common languages, like C and its derivatives, you can probably get away with
invoking `sudo dnf group install -y "development tools"` or `sudo apt-get
install build-essentials`.

But what if you need a different version of a toolchain? You don't want to
override what you have installed system-wide?

Normally this answered is solved by some kind of virtualization, all solved by
several types of hypervisors or container runtimes (yes, I am using
virtualization in an extremely broad sense).

You spin up a workspace VM or some workspace container, install your tools and
call it a day.

But that can be clunky. Personally I can find it to be a bit clunky. I like how
I have my editor set up, but editing files just to have then re-build your
"dev-environment" container, then build your app, then test it (or in the case
of the VM, ensure the directory is mounted onto the virtual machine with the
right permissions, ssh-ing or executing some remote command, blah) can get
clunky. I tend to give up on this segmentation approach pretty quickly when I am
just hacking along.

There is a better way.

Full disclosure, I am not an expert. I am also currently using a lot of
universal-blue flavored "distributions", and following what they deem to be
best-practices from a system-administration standpoint.

One of their (Universal Blue) beliefs is that installing packages through the
system package manager (dnf5) should be a last resort for the most part. But not
for any overwhelmingly "negative" reason so-to-speak. dnf5 actually installs
packages in an "atomic" way on these new immutable systems, through [package
layering](https://docs.fedoraproject.org/en-US/iot/adding-layered/), so its
relatively safe to install packages this way.

The caveat here is that because the filesystem is immutable, you now reboot into
the newly created, queued-up image. If anything goes wrong (like if your desired
package overwrites a config file) you can leverage the `rollback` feature of the
rpm-ostree to restore your system to the previous state. In fact, you can
even `pin` multiple rollback points -- kind of like git tags.

The problem arises from the following consequences: once you install a package
via dnf (rpm-ostree) you create a customized, derivative image *over* your
current system. I like to make the terrible analogy that this is akin to
managing a patch or a set of stashed commits over a git tree -- one that you'll
have to carefully and continually rebase over time.

Too many patches, and you've got a leaning tower of patch-pisa that may conflict
with future updates. Let it grow and you find yourself in rebase hell.

Remember installing twenty different toolbars on IE7 on your XP machine. Yeah.

So Universal Blue "distros" *really* do treat installing packages via
dnf/rpm-ostree as a last resort. Questions regarding mutation of the base
images, while probably necessary at times, are questions that are met with a
series of strongly opinionated alternatives.

The alternatives? For graphical apps, use sandboxed application packaging methods:
Flatpaks, Appimages, etc. Flatpaks are as easy to install as using the GUI
"software center" on many distros.

For command-line tools, they recommend Homebrew. If you'd like to run a service,
such as ngnix, adguard, etc, it is recommend to use
[Quadlets](https://docs.bazzite.gg/Installing_and_Managing_Software/Quadlet/)
(which are their own, really really cool thing, that I encourage you to check
out) -- TLDR: systemd service files but with containers!

In fact, Bazzite, one of the many "spins" of Universal Blue, (I really can't help myself
with the Fedora analogies here) actually "ranks" package formats from order of
most-to-least recommended for [daily
use](https://docs.bazzite.gg/Installing_and_Managing_Software/) to help with the
"order of precedence" for installing packages.

Ok, that's great and all...

But, if you're a Fedora user like myself, you'll find that you may have been using
Flatpaks for quite some time now, so that isn't *too* much of a big deal.

And Homebrew? The MacOS package manager?

But what about that one tool/app that is out of reach, is not well-packaged,
isn't in a standard repository? What about my precious Coprs?

These questions left me somewhat dismissive of the idea of an "atomic" immutable
distro, as introduced by Silverblue back in 2018-or-so. Being immutable in a
mutable system's world is... quirky.

For right now, let's at least tackle the Homebrew anomaly.

Jorge Castro, one of the people behind Universal Blue, does a much better job at
explaining the decisions behind
[Homebrew](https://www.ypsidanger.com/homebrew-is-great-on-linux/) than I ever
could, but I'm going to give it my best shot anyway. A lot of the tidbits about
Homebrew are actually taken directly from his article, so please, go give him a
read.

As a side note, Homebrew uses a lot of jargon. A lot of it is alcohol-brewing
related. It may make your head spin (or feel fuzzy?) Refer to the
[docs](https://docs.brew.sh/Formula-Cookbook#homebrew-terminology) here.

Homebrew on Linux, formerly referred to as Linuxbrew. From this point on, I will
refer to the Linux version of this package manager as `Linuxbrew` and the
general project/MacOS version as `Homebrew` to avoid confusion.

Anyway, Linuxbrew installs to the following *prefix*:
`/home/linuxbrew/.linuxbrew`

The short and skinny can be summated like so:

- Formulae are the definitions of how to install a package from upstream
sources.

- Kegs are the installation directory of a formula version (so
`$PREFIX/Cellar/$FORMULA_NAME/$VER`).

- Cellars are the directory which contains one or more racks (`$PREFIX/Cellar`).

- Racks are directories containing one or more kegs
(`$PREFIX/Cellar/$FORMULA_NAME`).

- Bottles are pre-built binary packages.

- Casks are *also* binary packages, usually reserved for the MacOS world, and
are for GUI applications.

- Taps are third-party repositories (kind of like Coprs, or the AUR),

- Brew Bundles are a declarative interface to Linuxbrew, where we can invoke
written `Brewfiles` to have reproducibility on our host systems.

Linuxbrew does *not* use any libraries provided by your host system, sans glibc/gcc (if
they are recent versions). On older systems, Linuxbrew installs current
versions of glibc/gcc. These are are self-contained into the install directory
(again, refer to the docs above!).

Alright, so why do we care about some esoteric MacOS package manager written in
Ruby?

Well, in 2021, Homebrew announced version
[3.1.0](https://brew.sh/2021/04/12/homebrew-3.1.0/?ref=ypsidanger.com) of their
beloved package manager. The big major change is that [Github Packages]() became
the default package {up, down}load location for both Homebrew and Linuxbrew.

The cool part is that these packages are just tagged OCI images, with open
analytics.

This makes it pretty easy to install system-wide, global command-line
applications that you **just gotta have** without package layering, and without
ham-fisting Flatpak to do something it really isn't meant to do.

And with Homebrew being a package manager with a pretty decent install base,
filled with users who enjoy shiny new things, it is pretty easy to *actually
get* the most up-to-date version of said shiny thing. Less time yak-shaving.

So great! We have a self-contained package manager that has shiny toys,
well-cached, and is pretty maintainable (and somewhat headache free?).

But again, that's nothing new, and it isnt anything that any user on any distro
can't just install and start using out of the gate. But it is a nice, somewhat
"ergonomic" alternative on an immutable distro -- and it doesn't manipulate the
base image.

It's a self-container, user-space package manager, that doesn't (and can't? --
to be confirmed) mutate your system packages.

"Alright man, you just spent like 500 words talking about Homebrew. What about
*my* devel environment?"

This is actually something I struggled with a bit. Remember that part about the
mental-model shift?

> But what if you need a different version of a toolchain? You don't want to
> override what you have installed system-wide?

I've worked on a couple of enterprise Python applications. Both new and old. On
my work system I have a smorgasboard of 7 different python version, venvs/
scattered about, direnvs for some projects, uv for others... blagh. It sucks
having to hop on and help out someone with their python sideproject, just to
have to either `pip install` yet-another python tool, or packaging utility or
whatnot.

You may ask:

"Why not use pyenv, uv, and ruff, and be done with it?"

I could. I have. I switched laptops at work and forgot to set it up again. Call
me lazy (or distracted) but I really can't be bothered to just set it all up
again, every time and then stubbornly refuse to update my vim config to
accomodate a new linter, or whatever. Also, in the short time that I've been
working with some python applications, there is pretty much no guarantee that
whatever "new" project that is following best-practices will be using direnv,
uv, ruff, tox, nox, sox... leaving me to with like, 30 different python tools.

"Well, you could just use vscode..."

I could. I really really could. I actually used to use it religiously in college
after Github was acquired by Microsoft, ending my use of Atom. But VSCode
doesn't solve the probem of needing to install the necessary runtimes on my host
system.

So here we are -- globally `pip install`-ing and sometimes `npm install -g` for
other useful CLI tools or LSP servers that don't live in many repos (stuff like
Gemini CLI or Claude Code), ending up with me having to maintain a hefty amount
of cruft on my system.

My knee-jerk reaction would be to `brew install $PKG`, but that doesn't solve
the aforementioned problem I find myself running into often -- do I really want
all of those tools globally? do I really need that? Not really. I typically only
spin these things up within a particular set of working directories.

Thankfully, we can have clean, logical separation and export binaries and
full-on graphical applications with `distrobox`'s built-in
[distrobox-export](https://distrobox.it/usage/distrobox-export/).

This knwowledge, paired with this fantastic blog post by Artem "Hackeryarn"
Chernyak: ["Distrobox in practice"](https://hackeryarn.com/post/distrobox/) and
Jorge Castro's own mad-scientist combination of
[chezmoi](https://www.ypsidanger.com/making-the-most-out-of-distrobox-and-toolbx/)
and [custom, signed, distrobox
images](https://github.com/ublue-os/boxkit?ref=ypsidanger.com) --as well as some
good [default](https://github.com/ublue-os/toolboxes?ref=ypsidanger.com) base
images-- really expanded the idea of "logical separation" of my working
environments. With distrobox, you can define separate $HOME dirs per container!

So now, I can have a git repository with each branch describing the *minimal*
editor config, and minimal packages/tools/toolchains needed for said development
environment on a per-workspace level. No more need to shim my global $PATH in my
host system's .bashrc, and no need to carry a ton of cruft in a massive vim
configuration.

Python project? Depending on the version, and "flavor" of the repo, I can have a
minimal distrobox container with `ruff, uv, direnv, pyenv, tox` and another with
`conda, black, hatch` and so on.

And if you don't feel like doing this... then whatever! If you don't specify a
separated $HOME, you just get your default homedir forwarded anyway.

And the same goes for pretty much everything else.

Ultimately, I've taken this opportunity to realize that we *should* be treating
our the base-image, operating-system level of our machines with the goal of
minimal-config-drift in mind.

We should be minimizing the amount of system-level packages which mutate said
system (if possible). We really should be using small scratch containers for our
development work. We should be using declarative configs for user-packages for
deterministic systems. We really ought to approach how we manage our day-to-day
machines in a similar manner to a production environment -- because these are
our production environments.

Anyway, enough rambling, here's how I approach my current install.

GUI Applications: Flatpaks.
System management: `ujust` (built into Universal Blue).

CLI Applications fall into two categories:

1. Global, utilitarian, easy?
    - `brew install $PACKAGE`

(as a bonus, you can dump all your packages into a ~/.Brewfile if you wish to just
start over from scratch! `brew bundle dump --global --force --describe`).

2. Anything else?
    - `distrobox --bin $BINARY_ABS_PATH` inside distrobox container.

System services:
[Quadlets](https://docs.bazzite.gg/Installing_and_Managing_Software/Quadlet/)
with the app container of my choice in
`~/.config/containers/systemd/$SERVICE_NAME`
then...
``` bash
systemctl --user daemon-reload
systemctl --user start $SERVICE
```
