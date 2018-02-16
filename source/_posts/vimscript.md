title: Vimscript the terrible way
date: 2014-1-31
tags: Vim
---

{%img /images/Photo.jpg%}
source: [pixiv](http://www.pixiv.net/member_illust.php?mode=medium&illust_id=41290048)

> Exercises: Drink a beer to console yourself about Vim's coercion of strings to integers.
>                                                -- Steve Losh, Learning Vimscript the hard way

Vim is a superb and classy text-editor, but VimScript, well, is far from a usable scripting language.

Vimscript is extremely powerful, but has grown organically over the years into a twisty maze ready to ensnare the unwary who enter it.
Options and commands are often terse and hard to read

On the surface, VimL looks like an agglomeration of Lua, Perl and primitive Vim configuration language. A litany of reserved words and special operators, along with Regex line-noises, gives rise to yet another write-only language. VimL supports both function call and primitive command, which seasons VimL with one more layer of complexity and inconsistency.

This is culpable, but is not the most irremediable obfuscation of VimL. The tittle should belong to event system. Vim executes one and only one handler for an event once a time. To bind multiple handlers to one event, users have to write their own [custom function](http://vim.wikia.com/wiki/Overload_a_key_with_multiple_handlers).

It seems very easy for anyone, but at field, handling Vim plugin conflicts is a Herculean task. Vim comes out with many plugins loaded, and users' .vimrc are liable to frequent editing. Maintaining a list of event handlers requires a user to face with plugins' source code(verbose namespace, illegible syntax, Byzantine structure, or line noises fraught with all the aforementioned three). And this will be users' chronic concern since Vim's configuration are constantly changing. What's more, VimL's events are not sufficient. Until recent that  Vim 7.3 adds InsertPre event, there is no event for pressing arbitrary key. Users have to write recursive function to capture [all keys](http://vim.wikia.com/wiki/Capture_all_keys)

Lastly, there is little VimL material users can find. Vim's help doc is the only comprehensive and up-to-date document, but it is awkwardly organized. Vim community also favors Python over VimL, indicating that VImL will have [less chance](http://www.vim.org/sponsor/vote_results.php) to get enhancement. Over the internet, few complaints and swearings about VimL are heard. It shall be interpreted as few VimL developers instead of few VimL defects.

