title: JavaScript Error tracking in browsers
date: 2015-11-17
tags: JavaScript
---

Error tracking is one of the awful part of JavaScript. Server side error tracking requires several configuration, which is not hard because server is under developers' full control, after all. For Android/iOS applications, error tracking is integrated into integrated into platform framework. Unfortunately, error tracking in browser is like survival in wild jungle.

Here are some common pitfalls.

Incompatible API
===

JavaScript errors in different browsers have different field names, as usual.

One should never bother to care about api incompatibility among browsers. Here is the snippet to normalize those quirks.
Curse the variable `ieEvent`.

```javascript
function errorTracker (err) {
  var ieEvent
  if (typeof err !== 'object') {
    ieEvent = {}
    err = window.event
    ieEvent.message = err.errorMessage
    ieEvent.filename = err.errorUrl
    ieEvent.lineno = err.errorLine
    err = ieEvent
  }

  return {
    message: err.message,
    line: err.lineno || err.lineNumber,
    file: err.filename || err.fileName
  }
}
```

CORS Header
====
You will see a lot of `Scritp error` message in your CDN hosted javascript file. Browsers report this useless error intentially. Revealing error details to web page in different domain is a security issue. It can leak one's privacy and helps phishing and social engineering. To quote the [SO answer](http://stackoverflow.com/questions/5913978/cryptic-script-error-reported-in-javascript-in-chrome-and-firefox)

> This behavior is intentional, to prevent scripts from leaking information to external domains. For an example of why this is necessary, imagine accidentally visiting evilsite.com, that serves up a page with `<script src="yourbank.com/index.html">`. (yes, we're pointing that script tag at html, not JS). This will result in a script error, but the error is interesting because it can tell us if you're logged in or not. If you're logged in, the error might be `'Welcome Fred...'` is undefined, whereas if you're not it might be `'Please Login ...'` is undefined. Something along those lines.

And in Chromium's [source code](https://chromium.googlesource.com/chromium/src.git/+/master/third_party/WebKit/Source/core/dom/ExecutionContext.cpp#178]), we can see error is sanitized, if the `corsStatus` does not satisify some condition.

```cpp
bool ExecutionContext::shouldSanitizeScriptError(const String& sourceURL, AccessControlStatus corsStatus)
{
    if (corsStatus == OpaqueResource)
        return true;
    return !(securityOrigin()->canRequestNoSuborigin(completeURL(sourceURL)) || corsStatus == SharableCrossOrigin);
}

bool ExecutionContext::dispatchErrorEvent(PassRefPtrWillBeRawPtr<ErrorEvent> event, AccessControlStatus corsStatus)
{
    EventTarget* target = errorEventTarget();
    if (!target)
        return false;

    RefPtrWillBeRawPtr<ErrorEvent> errorEvent = event;
    if (shouldSanitizeScriptError(errorEvent->filename(), corsStatus))
        errorEvent = ErrorEvent::createSanitizedError(errorEvent->world());

    ASSERT(!m_inDispatchErrorEvent);
    m_inDispatchErrorEvent = true;
    target->dispatchEvent(errorEvent);
    m_inDispatchErrorEvent = false;
    return errorEvent->defaultPrevented();
}
```

To enable `SharableCrossOrigin` scripts, one can add `crossorigin` attribute to script tags, and, add a `Access-Control-Allow-Origin` head in scripts' server response, just like cross orgin XHR.

Like this.
```html
<script src="http://somremotesite.example/script.js" crossorigin></script>
```

And in your server conf, say, nginx, add something like this

```conf
server {
  server_name static.jscdn.com;
  # blah blah blah

  location ~ \.js {
    add_header Access-Control-Allow-Origin "*";
  }
}
```

More Browser Quirks and nasty ISP
====

Modern browser will protect users' privacy and respect developers' CORS setting. But IE may screw both. In some unpatched Internet Exploers, all script errors are accessible in `onError` handler, regardless of their origins. But some Internet Explorers, patched, just ignore the CORS head and swallow all crossorigin error messages.

To catch errors in certain IEs, developers must manually wrap their code in `try {...} catch (e){report(e)}` block. Alternatively, one can use build process to wrap function, like [this](https://github.com/BetterJS/try-wrapper).

[Zone](https://github.com/angular/zone.js) should also be a good candidate for error tracking, and does not require build process. Though I have not tried it.

Another issue in error tracking is ISP and browser extensions. `onError` callbacks will receive all error in the host page. It usually contains many ISP injected script and extension script which trigger false alarm errors. So wrapping code in `try ... catch` may be a better solution.


**UPDATE:**

It seems `Zone` like hijacking method has been used in error tracking product. Like [BugSnag](https://bugsnag.com/blog/js-stacktraces). The basic idea is: If code is executed synchronously, then it can be `try ... catch ...`ed in one single main function. If code is executed asynchronously, then, by wrapping all function that takes callback, one can wrap all callbacks in `try ...catch ...`.

```javascript
function wrap(func) {
    // Ensure we only wrap the function once.
    if (!func._wrapped) {
        func._wrapped = function () {
            try{
                func.apply(this, arguments);
            } catch(e) {
                console.log(e.message, "from", e.stack);
                throw e;
            }
        }
    }
    return func._wrapped;
};
```
The above code will wrap all `func` in try and catch. So when error occurs, it will be always logged. However, calling wrapper function in every async code usage is impractical. We can invert it! Not wrapping callbacks, but wrapping functions that consume callbacks, say, `setTimeout`, `addEventListener`, etc. Once these async code entries have been wrapped, all callbacks are on track.

And, because JavaScript is prototype based language, we can `hijack` the `EventTarget` prototype and automate our error tracking code.

```javascript
var addEventListener = window.EventTarget.prototype.addEventListener;
window.EventTarget.prototype.addEventListener = function (event, callback, bubble) {
    addEventListener.call(this, event, wrap(callback), bubble);
}
```


IE9 and friends
=====

Sadly, IE does not give us stack on error. But we can hand-roll our call stack by traversing `argument.callee.caller`.

```javascript
// IE <9
window.onerror = function (message, file, line, column) {
    var column = column || (window.event && window.event.errorCharacter);
    var stack = [];
    var f = arguments.callee.caller;
    while (f) {
        stack.push(f.name);
        f = f.caller;
    }
    console.log(message, "from", stack);
}
```


Garbage Collector Issue
=====
Error reporting is usually done by inserting an `Image` of which the `url` is the address of logging server comprised of encoded error info in query string.

```javascript
var url = 'xxx';
new Image().src = url;
```

But the `Image` has no reference to itself, and JS engine's garbage collector will collect it before the request is sent. So one can assign the `Image` to a variable to hold its reference, and withdraw the reference in the `onload/onerror` callback.

```javascript
var win = window;
var n = 'jsFeImage_' + _make_rnd(),
  img = win[n] = new Image();
img.onload = img.onerror = function () {
  win[n] = null;
};
img.src = url;
```
