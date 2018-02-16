title: Updated Ctags for TypeScript
date: 2015-10-10 18:26:14
tags: TypeScript
---

TypeScript has a decent tooling chain. However, sometimes a simple `ctags` is more handy and efficient.

Here is a updated ctag configuration for TypeScript. It supports more features in TS 1.5+.


```
--langdef=typescript
--langmap=typescript:.ts
--regex-typescript=/^[ \t]*(export)?[ \t]*class[ \t]+([a-zA-Z0-9_]+)/\2/c,classes/
--regex-typescript=/^[ \t]*(export)?[ \t]*abstract class[ \t]+([a-zA-Z0-9_]+)/\2/a,abstract classes/
--regex-typescript=/^[ \t]*(export)?[ \t]*module[ \t]+([a-zA-Z0-9_]+)/\2/n,modules/
--regex-typescript=/^[ \t]*(export)?[ \t]*type[ \t]+([a-zA-Z0-9_]+)/\2/t,types/
--regex-typescript=/^[ \t]*(export)?[ \t]*namespace[ \t]+([a-zA-Z0-9_]+)/\2/n,modules/
--regex-typescript=/^[ \t]*(export)?[ \t]*function[ \t]+([a-zA-Z0-9_]+)/\2/f,functions/
--regex-typescript=/^[ \t]*export[ \t]+var[ \t]+([a-zA-Z0-9_]+)/\1/v,variables/
--regex-typescript=/^[ \t]*var[ \t]+([a-zA-Z0-9_]+)[ \t]*=[ \t]*function[ \t]*\(\)/\1/l,varlambdas/
--regex-typescript=/^[ \t]*(export)?[ \t]*(public|private)[ \t]+(static)?[ \t]*([a-zA-Z0-9_]+)/\4/m,members/
--regex-typescript=/^[ \t]*(export)?[ \t]*interface[ \t]+([a-zA-Z0-9_]+)/\2/i,interfaces/
--regex-typescript=/^[ \t]*(export)?[ \t]*enum[ \t]+([a-zA-Z0-9_]+)/\2/e,enums/
```

For more vim integration, you can give [YATS](https://github.com/HerringtonDarkholme/yats.vim) a look
