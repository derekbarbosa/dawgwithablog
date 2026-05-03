---
title:  Some distrobox hacks
date:   2026-05-02 9:35:00
categories: [distrobox]
tags: [containers, distrobox, hacks, ruby, jekyll]
---

So I've been awfully radio silent, and that's OK! Life is just one of those
things where it seems like you're always chasing a runaway wagon. At least, for
me it seems that way.

At work, we've been told/mandated to use the corporate standard builds, so that
definitely put a pause on my "Immutable Everywhere!" ranting to my colleagues
for now.

> So, Derek, why the long pause?

Well, I didn't have much to write about that felt _worth it_ -- and with
everyone big on LLMs promising everyone will be out of a job being *the thing*
to write about, I felt like writing about fun little tools was... pointless?

> But you only wrote one entry...

Yeah. Good point. Let's drop the doomer act, here's another.

Let's talk about Distrobox. I enjoy the "wrapper" it provides over a container
runtime; it eliminates the needs for some hacks to get "interactive" container
sessions working well. The idea of a small, separate, rebuildable (and
maintainable) configuration of tools for a development environment still is
nifty. In fact, I have been religiously using `toolbx` containers at work for
several years now -- mainly to deal with python packaging for some small web
services that I occasionally hack on.

You can actually see this [horrendous
hack](https://github.com/derekbarbosa/dotfiles-stow/blob/work/bash/.bashrc) I
employ in my work config's .bashrc

```bash
if [ -f "/run/.toolboxenv" ] && [ -f "/run/.containerenv" ]; then
▎ eval "$(direnv hook bash)"
▎ unalias vim
▎ export EDITOR='vim'
fi
```

But, I have been having a bit of an issue with some personal machine's configs.
In particular, I've been experimenting with a concept I referenced in my last
post -- isolating home directories from your working, base $HOME

> And if you don’t feel like doing this… then whatever! If you don’t specify a
> separated $HOME, you just get your default homedir forwarded anyway.

Yeah, that works well a good deal of the time, but funnily enough, trying to
install the deps and build my site seems to be totally borked:

```bash
$ bundle install
There was an error while trying to create `/home/derek/.distrobox/homes/blog/.local/share/gem`.
It is likely that you need to grant executable permissions for all parent directories and write permissions for `/home/derek/.distrobox/homes/blog/.local/share`.
```

A head scratcher.

I set my new $HOME dir to "$HOME/.distrobox/homes/blog",
chmod'ing the `.local` directory from the `init_hook`s in distrobox didn't seem to
do the trick, and neither did a quick:

```bash
mkdir -p ${HOME}/.local/share/gem
```

In fact, invoking $HOME within the `init_hooks` references the $HOME dir of the
"base" system. You definitely don't want to be screwing around with chmod in
that case, at least, with those variables.

My solution? Well, I ignored it for a while (which is partly why I didn't bother
with writing a new entry in so long).

After some time, I resolved to referencing the entire path in my `init_hook`s in
`mkdir -p`, and manually performed a `chmod -R ugo+rwX .local` once I entered
into the `blog` container. I _could've_ done that in the pre-init hooks, but I'm
keeping it as a mental note for now.

After all that? `bundle install` && `bundle exec jekyll serve` work as expected!

So, a note for future reference, if you also have been hitting snags in the
"isolated homedir" ethos, check your directory permissions. Maybe this is a
lesson in using Jekyll in 2026. Thanks, Github Pages.

I should really transition to just a simpler markdown -> html SSG.
