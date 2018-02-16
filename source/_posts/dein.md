title: Trying out dein.vim -- a dark powered plugin manager
date: 2016-02-26
tags: Vim
---

Dark Powered Plugin
---
[Original Japanese post here](http://qiita.com/yoza/items/2f8bd33a18225754f346)

Vim already has a lot of plugin managers.
But our [Dark Vim Master](https://twitter.com/ShougoMatsu) has released neobundle's successor, a brand-new plugin manger called [dein.vim](https://github.com/Shougo/dein.vim).

> Dein.vim is a dark powered Vim/NeoVim plugin manager.

To put it in plain English, dein.vim is a plugin manager focusing on both **installation performance** and **startup performance**.
It looks like a neobunle with vim-plug's speed, or a vim-plug with neobunle's feature. The best of both worlds.

In this blog post I will first show minimal configuration for dein.vim, and then try to explain the dark power of dein, if you are interested in what makes dein.vim so fast.

Minimal Configuration
---

> Though dark powered, dein.vim supports vim and neovim.

Sadly, there is no installation script for dein.vim for now.
So let's manually install it.

First, clone dein.vim's source.

```bash
mkdir -p ~/.vim/dein/repos/github.com/Shougo/dein.vim #recommended path
git clone https://github.com/Shougo/dein.vim.git \
    ~/.vim/dein/repos/github.com/Shougo/dein.vim
```

If you are familiar with neobundle/vundle, you will find dein.vim's path so different. It is because dein uses a new approach to manage plugin's source.

Optionally, you can backup your vimrc for profiling, as I will show later.

Then, in your `init.vim` or `.vimrc`.
```vim
set nocompatible
set runtimepath+=~/.vim/dein/repos/github.com/Shougo/dein.vim " path to dein.vim

call dein#begin(expand('~/.vim/dein')) " plugins' root path
call dein#add('Shougo/dein.vim')
call dein#add('Shougo/vimproc.vim', {
    \ 'build': {
    \     'windows': 'tools\\update-dll-mingw',
    \     'cygwin': 'make -f make_cygwin.mak',
    \     'mac': 'make -f make_mac.mak',
    \     'linux': 'make',
    \     'unix': 'gmake',
    \    },
    \ })

call dein#add('Shougo/unite.vim')

" and a lot more plugins.....

call dein#end()
```

Fire up your neovim/vim, call dein.vim's installation function

```vim
:call dein#install()
```

Wait and brew yourself a cup of (instant) coffee.
Then you can confirm your installation, for example call `:Unite dein`

More features
---

The most important feature of dein.vim is lazy load. Here are some typical usages worth mentioning.

```vim
" lazy load on filetype
call dein#add('justmao945/vim-clang',
      \{'on_ft': ['c', 'cpp']})

" lazy load on command executed
call dein#add('scrooloose/nerdtree',
      \{'on_cmd': 'NERDTreeToggle'})

" lazy load on insert mode
call dein#add('Shougo/deoplete.nvim',
      \{'on_i': 1})

" lazy load on function call
call dein#add('othree/eregex.vim',
      \{'on_func': 'eregex#toggle'})
```

The last two lazy loading conditions are not available in Vimplug. While lazy loading according to mode change is very convenient.

Two Tales of Plugin Manager
---

From here on I will talk about dein's internal feature. It's my personal observation, please pardon my mistake and misunderstanding.

Vim's plugin managers have to take two aspects into consideration: installation time and startup time.
You can read [junegunn](https://github.com/junegunn/)'s blogs article, [this](http://junegunn.kr/2013/09/writing-my-own-vim-plugin-manager/) and [this](http://junegunn.kr/2014/07/vim-plugins-and-startup-time/), for more details on plugin manager.

> dein.vim is fast because it uses dark power

[JK](http://www.urbandictionary.com/define.php?term=jkjk)[JK](https://en.wikipedia.org/wiki/JK). Actually, dein.vim optimizes both two performance, by following measures:

1. **parallel installation** dein.vim uses either [vimproc](http://exhentai.org/s/56aedaf7ad/908231-9) or neovim's [async job](https://neovim.io/doc/user/job_control.html) to download plugins concurrently.

2. **precomputed runtimepath** dein.vim will copy all plugins' subdirectory into a cache directory. This merges all runtime VimScript files into one directory. So dein doesn't need to compute runtimepath on startup.

Dein.vim also ditches command usage in favor of function calling, which may also contribute to performance(I'm not sure, though).

Troubleshooting
---

Because dein.vim uses parallel processing, when errors occur in installation it may be hard to figure out which plugin went wrong. Usually you can see the error messages by `:message` after all plugin finished fetching (regardless of success or not). Or you can use `dein#check_install` function.

Also, precomputed cache makes modifying plugin harder. You will need to call `dein#recache_runtimepath()` after modification. This also applies to disabling plugins.

Lastly, if you happen to live in a country with stupid censorship which prevents github access. You will need a proxy and set `g:dein$install_process_timeout` to a larger value.

For more info please refer to [doc](https://github.com/Shougo/dein.vim/blob/master/doc/dein.txt).

Profiling dein's power
---

Profiling vim's startup time has a rather [standard method](http://usevim.com/2012/04/18/startuptime/). You can try it by backup old vimrc and compare it to dein.

```bash
nvim -u old-init.vim --startuptime neobundle.log # or change nvim to vim
nvim -u new-init.vim --startuptime dein.log
```
From the log, I can see dein.vim gives me 20ms startup boost! Mainly from boosting `sourcing vimrc`. Amazing!

If you use a lot of plugins, dein is definitely a must-try!
