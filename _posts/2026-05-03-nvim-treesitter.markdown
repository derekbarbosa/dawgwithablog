---
title:  Migrating Treesitter to v0.12
date:   2026-05-03 09:15:00
categories: [editors]
tags: [neovim, editors, treesitter]
---

TLDR: For migrating your tree-sitter configuration to one that is nvim-v0.12
compatible, I would recommend replacing `tree-sitter.nvim` with romus204's
[tree-sitter-manager](https://github.com/romus204/tree-sitter-manager.nvim)
plugin.

This will be a somewhat subjective blog post.

I love vim.

For me, it sits perfectly in between the "pure, lightweight text editor" and
"Jetbrains IDE" spaces. I think it is a really pure example of how knowing your
tools can unlock a great deal of productivity.

I love having windows into buffers, and organizing workspaces with tabs. Throw
some tmux into the mix and __chef's kiss__

When working in a remote box somewhere, the only thing I find myself missing are
some fancier ripgrep plugins that I have mapped to some fancy keybinds.

In some cases, I find mostly myself _viewing_ text-based files with some
combination of:

```bash cat somefile | less ```

and reaching for vim when I have to perform all sorts of edits.

and that works a great deal of the time.

And that's not to say I _don't_ like using IDEs, it is probably just something
broken within my own brain that prevents me from extracting their maximum value.

I would say that I use them *all the time* for editing TS/JS in some frontend
projects, but I fear that saying:

> "I use Codium with a couple of plugins"
>  - me, probably

is still far, far off from what most consider a fully-fledged, pre-packaged,
batteries-included IDE experience and actually closer to the text-editor world
that I am so dearly accustomed to.

I spend a great deal of my time in the terminal. So, for me, I find it
imperative that something can be quickly spun-up/shut-down, restored to some
previous state, and generally gets out of my way.

To maintain some level of focus -- if I am not using it, it is generally closed,
or shunted off to some other workspace or monitor. In the case of IDEs, I
generally find myself closing them and donating whatever memory they consumed to
the copious amount of browser tabs I have open.

Frankly, I think that's what bothers me about things like Visual Studio (which,
I really enjoyed using in my short stints with C#). The terminal is subjected to
small, pitiful pane; the padding on the menu dialogs and sidebars are awfully
huge; the UI seems to overshadow the two most important elements to begin with:
your code and it's output (generalizing here).

Those are all minor nitpicks and gripes, and I realize that there is nothing
preventing me from just using my WM to pane my IDE on one-half of my monitor,
and my terminal in the other.

But it still seems to me like there is a lot of "wasted space" in a
mouse-focused GUI when compared to something that is entirely keyboard driven.

> Just get better at the keyboard shortcuts in VS?

I salute you, because for the life of me, learning the mappings in a modal
editor seemed simpler than some of the keybindings in VS. This is also a salute
to you Photoshop Gurus, you all deserve to be in MENSA or something.

Ok, now onto the real reason for this blog post.

I _like_ nvim (neovim).

> What's the difference?

While many like to equate vim/nvim as a mutt/neomutt situation, I tend to view
neovim (neovim) as something closer to emacs than it is to vim. Just hear me
out.

Neovim maintains a great deal of backwards compatibility with vimscript plugins
and via the vim api. However, over the years, with the addition of a newer DSL
for vim, and changes to the Lua APIs in neovim, it is becoming apparent that it
  is primarily a "best effort" scenario rather than a promise, and that isn't
  explicitly a "bad thing".

> Why do you "like" it rather than love it?

There are a ton of things that make the development experience in neovim great.
For example: I love telescope, a lot, like seriously. I've found it to be one of
the best "plugins" aimed at developer experience that I've ever used across a
great deal of the IDEs that I have tried.

And frankly, I enjoy the aspect of getting "more out of the box" that is
configurable with a sensible language. Built-in "opt-in" configuration, that can
be as minimalist or maximalist as you'd like it is always a "good thing" IMO.

What I don't love, funnily enough, is the growing pain of using something that
cannot promise stability in it's major version upgrades. I can deal with that
for the extensibility and "new shiny toys".

It can be annoying however, when the internal API for something like
tree-sitter, a parser generation tool, changes completely, breaking things like
the nvim-treesitter plugin, which have been around for quite a while.

It can be pretty poor UX when you have to:

- Remove all your LSP servers (:TSUninstall all)
- Delete the plugin and "clean" it from your registered/cached plugins
- Comment out your config

To prevent a copious amount of errors on your next "init".

Thanks to some google-fu and browsing on webforums (IOW, reddit, who am I
kidding?) -- I found a great plugin that manages your treesitter queries akin to
mason-nvim and nvim-lspconfig.

[tree-sitter-manager](https://github.com/romus204/tree-sitter-manager.nvim)
appears to do just that, and it seems that (so far) you can drop most of your
pre-existing config there. So after a half-hour or so of reading, you can then
fiddle with your config, and pretend it works until it doesn't.

> You sound like you hate this thing.

It's an interesting dillema, similar to the 0.10 to 0.11 upgrade, which
completely changed the internal API for LSPs and code diagnostic servers --
breaking things like null-ls, leading to it's eventual deprecation. When the
core project "absorbs" your functionality and maintenance burden, what becomes
of your project? Should people even develop plugins for your "extensible"
project if you cannot promise them some level of API stability?

Neovim seems to play in this weird middle ground, nothing quite like Rust's
microproject ecosystem -- leading to a slew of fragmented, archived,
unmaintained packages squatting on crates.io -- but it isn't a kitchen sink of
functionality and features (yet?).

It thrives on community-driven plugins, for now, but even plugin managers like
Lazy.nvim are being supplanted for the nvim's shiny new default plugin manager.
How much longer until plugins are relegated purely to theming, workspace
organization, etc? And would that be a "bad" thing?

I don't have an answer or solution as to how you sensibly "grow" a project, hit
your milestones, and improve the UX without breaking things and accruing _more_
technical debt-- but that is primarily the reason why nvim has a planned
[roadmap](https://github.com/neovim/neovim/issues/20451) to 1.0, which involves
painstaking attention to detail to an "API contract".

However, it wouldn't be totally inappropriate to say that this is akin to a
"Ship of Theseus" or some euphemism that describes the paradoxical phenomenon of
replacing all of your components to deliver the same "goal".

But would I go back to plain-old-vim with COC and a big honkin' vimscript config
if I wanted an editor with a little more *pizzaz*?

No.

But it really does have me considering what I should have as a good "fallback"
of sorts if I _need_ some stable, near-frozen configuration that works. It is a
good thought experiment on the intersection of:

- What you need
- What you want
- What you're willing to put up with

It may be wise to unalias nvim as vim from now on, and to maintain a separate
"light" configuration. A "break-in-case-of-emergency", if you will.
