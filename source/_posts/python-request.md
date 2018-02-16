title: Request is a good lib
date: 2014-2-2
tags: Python
---

{%img /images/python-request.jpg%}
source: [pixiv](http://www.pixiv.net/member_illust.php?mode=big&illust_id=41320823)


Web scrapping is a common task for script languages like python.
Yet Python standard libraries provide twisted utility to couple with this simple task. `urllib[23$]` ruins my python newbie days.
To achieve cookie support, you have to import cookielib, create a new cookie jar via the factory method, subclass a opener from the urllib, and finally use the forged opener to make a request.

Bloated boilerplate code does not seem pythonic at all. We want a simple scrapping utility that provides concise API like http verbs and, preferably, automatic session management. **Request** is the library comes to rescue.

The structure of Request:

* model
* session
* adapter
* api
* etc...

`model` irons response and request parameters into unified objects. Both `Request` and `Response` supports generator style and file style. Magic methods empowers `Request` pythonic syntax. The most dirty works(say, encoding stuffs) lies here.

`session` acts as controller in  MVC design. It combines adapter, which sends request, authentication and cookie session. Request's streamlined API stems from an interface mocking real browsers.

`adapter` is a wrapper of urllib3, supporting connection pool management, `keep-alive` request and proxies.

`api` is just an alias for `1. open session 2. prepare request 3. send request`. And all other modules are helpers that do stuffs like encoding and making auth token.

The design philosophy under Request is : `Simple is better than functionality`, however, it still grants users plenty of features.
Specific as the library aims to be, the philosophy underlying it applies much wider. jQuery's almighty $, Chrome's omnipotent omnibox, Google's preeminent search engine and etc. Simplicity rocks, Usability counts.
