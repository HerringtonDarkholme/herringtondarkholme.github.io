title: magic HTML.js
date: 2014-05-17
tags: JavaScript
---

{%img /images/html.jpg%}
source: [pixiv](http://www.pixiv.net/member_illust.php?mode=medium&illust_id=43531055)

HTML.js is a library full of syntactic sugar. It changes html elements dynamically so that the methods reflect their children node. For example, code like HTML.body.header.hgroup.h1 utilizes chain methods to mirror the structure of dom.

ES5 `Object.defineProperty` and `mutationObserver` conjure up the magic. HTML.js provides an eponymous HTML api object initialized by an internal `node` method, which add all _tag_ methods to its argument object. All _tag_ methods are defined by `Object.defineProperty` with `get` option. So tag methods behave like getter methods: every time user access these attributes, tag methods return _HTMLifyed_ elements that are ready to be chained (HTMLifyed elements are normal HTMLElements that have been extended by the internal node method mentioned above).

Enable tag methods responsive to dom manipulation, HTML opts for mutationObserver to keep an eye on the root element. Once elements have been changed, mutationObserver detects the change and notifies HTML to refresh methods of the corresponding elements.

However, syntactic sweetheart fails to belie some design deficits and practical problems in this library. `Getter` methods abstain legacy browsers that still occupy about 10% market share. MutationObserver itself is not that horribly slow, but registering a watchdog on <html> is almost certainly a performance killer for massive dom manipulations.

But the most notorious code smell comes from yet another place, a pure design decision that functions returning either element or array. It is certainly one of the most sloppy practice in dynamic language. In static typed language those function can only have return type `Any`. Surely this is not informative and bothers user to take the risk of casting results. Indeed, the author mentioned this on the [homepage](http://nbubna.github.io/HTML/) and tried defend this api design by the excuse of conditional context where users can avoid quandary. But a good library shall be as much care-free as possible. Providing api that returns one single element is probably better than  leaving the users to guarantee element's uniqueness. Ad-hoc polymorphism is determined by function arguments, not return type.

HTML's api reminds me of the keyword `null`.  Admittedly it is theoretically feasible to entrust programmers with the role of checking uniqueness/existence. But why ぬるぽ is still one of the most prominent haunting apparition in our code?
