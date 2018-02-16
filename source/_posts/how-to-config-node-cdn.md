title: A collection of nodejs CDN for Celestial Imperial
date: 2016-06-07
tags: nodejs
---

Living inside of the Great Fire Wall is a so onerous ordeal that survival guides can be compiled into an anthology.

Here is how to configure various popular open source project to use their CDN in Ch******na.

Plenty of nodejs project provide environment variable to configure mirror/CDN to download source code or binary.

nvm / nodejs
---
[Ref](https://cnodejs.org/topic/5338c5db7cbade005b023c98)

```bash
NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node nvm install 4
```

npm
---
[Ref](https://cnodejs.org/topic/5338c5db7cbade005b023c98)

```bash
npm --registry=https://registry.npm.taobao.org install koa
```

Kudos to [fengmk2](https://cnodejs.org/user/fengmk2)

PhantomJS
---

[Ref](https://cnodejs.org/topic/538ed941c3ee0b5820889f66)

```bash
PHANTOMJS_CDNURL=https://npm.taobao.org/dist/phantomjs npm install phantomjs --registry=https://registry.npm.taobao.org --no-proxy
```

Kudos to [fengmk2](https://cnodejs.org/user/fengmk2), again!

node-sass
---

[Ref](https://github.com/cnpm/cnpm/pull/76/files)

```bash
SASS_BINARY_SITE=https://registry.npmjs.org/node-sass npm install node-sass
```

Kudos, again and again, to fengmk2

Docker
----

[Ref](http://www.imike.me/2016/04/20/Docker%E4%B8%8B%E4%BD%BF%E7%94%A8%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F/)

Install Docker

```bash
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

Config accelerator

You need to register an account in [daocloud.io](http://xxxxxx.m.daocloud.io).

```bash
echo "DOCKER_OPTS=\"--registry-mirror=http://xxxxxx.m.daocloud.io\"" | sudo tee -a /etc/default/docker
sudo service docker restart
```

Kudos to daocloud.

Pypi
---
[Ref](http://www.cnblogs.com/meelo/p/4636340.html)

```bash
cat <~/.pip/pip.conf <<EOF
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
EOF
```

Kudos to aliyun


sbt
----
[Ref](http://freewind.in/posts/2619-sbt-global-repo/)

```bash
cat > ~/.sbt/repositories <<EOF
[repositories]
local
osc: http://maven.oschina.net/content/groups/public/
typesafe: http://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
sonatype-oss-releases
maven-central
sonatype-oss-snapshots
EOF
```

Kudos to [freewind](http://freewind.in/)

And these links is useful:
[加速 SBT 下载依赖库的速度](https://segmentfault.com/a/1190000002474507)
[how to make sbt jump over GFW](http://afoo.me/posts/2014-11-05-how-make-sbt-jump-over-GFW.html)

Gopm
----

[Ref](https://github.com/gpmgo/gopm)

It is quite clear in the doc and help command.

```bash

gopm get <import path>@[<tag|commit|branch>:<value>]
gopm get <package name>@[<tag|commit|branch>:<value>]

useful options:
   --update, -u		update package(s) and dependencies if any
   --gopath, -g		download all packages to GOPATH
```


Conclusion
---
Hail Aliyun! Hail fengmk2!
For the unfortunate who live in a crappy ignorant country.

{%img /images/Hatsune.Miku.full.2006986.jpg%}
