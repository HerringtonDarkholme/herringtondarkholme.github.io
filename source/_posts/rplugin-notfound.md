title: Trouble Shooting NeoVim's rplugin not found
date: 2015-12-06
tags: Vim
---

[NeoVim](http://geoff.greer.fm/2015/01/15/why-neovim-is-better-than-vim/) is awesome. But after its 0.1 release, neovim is not that awesome, after all, in a old vimmer's eye.

To support [XDG](https://wiki.archlinux.org/index.php/Xdg-open) configuration, NeoVim [changed default config paths](https://github.com/neovim/neovim/wiki/Following-HEAD#20151026). After that patch, neovim search `~/.config/nvim/init.vim` rather than our old friend `~/.nvimrc`. So I changed my zshrc to alias `v` to `nvim -u ~/.nvimrc`. So I can use old configuration without relocating files.

It works fine, except [Deoplete](https://github.com/Shougo/deoplete.nvim) always complain about its remote plugin is not registered. When I execute `UpdateRemotePlugins` as Deoplete's doc said, a new `.-rplugin-` file always spawn in my working directory. Without that weird file, Deoplete will never work.

I think this is configuration problem. But how can I figure out what happened? I start to resolving this by guess.

`.-rplugin` is the critical file on which Deoplete depends. So NeoVim should search for it when booting. I searched for NeoVim's repository for its usage. Yes, it does appear in neovim's source, in `neovim/runtime/autoload/remote/host.vim`.

It reads:

```vimscript
let s:remote_plugins_manifest = fnamemodify(expand($MYVIMRC, 1), ':h')
      \.'/.'.fnamemodify($MYVIMRC, ':t').'-rplugin~'
```

Hmmm, NeoVim will find `.-rplugin` in the same directory of `$MYVIMRC`. But where does `$MYVIMRC` come from?

Searching neovim's doc gives me the answer. In [`:h starting`](https://github.com/neovim/neovim/blob/84a5709a86cc8d2d90cc29eea26f254b9b5d85fa/runtime/doc/starting.txt#L394), it writes:

> If Vim was started with "-u filename", the file "filename" is used.
  All following initializations until 4. are skipped. $MYVIMRC is not
  set.

Oh, so I have to relocate my `vimrc` file. After that, every thing works :).
