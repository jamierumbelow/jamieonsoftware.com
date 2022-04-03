---
layout: post
title: "A quick note on web_accessible_resources in Safari Web Extensions"
date: 2022-04-03 22:30:04 +0100
tags: javascript
---

The Safari Web Extension mechanism is an important step forward for iOS apps – and a significant part of my current project, on which more soon – but the documentation is a little fragmented and the tooling isn't nearly solid enough just yet. Developing these new extensions is challenging.

A small contribution to changing that:

By default, extension files that aren't explicitly whitelisted in the `manifest.json` file are inaccessible from the browser. One common use case of the `content.js` script is to inject scripts into the same execution environment of the active web page. However, no such injection is possible unless the loaded file is whitelisted.

The `manifest.json` spec [defines](https://developer.chrome.com/docs/extensions/mv3/manifest/web_accessible_resources/) a `web_accessible_resources` parameter, which allows extensions to whitelist resources for access from the browser.

Thus, the following whitelist:

    "web_accessible_resources": ["images/foo.png"]

Allows you to generate the resource's URL with the following code:

    browser.runtime.getURL("images/foo.png")

Which gives you the following sort of URL:

    safari-web-extension://2AB33852-D69B-4ED6-99AD-A4839DFEC7ED/images/foo.png

The ID directly after the protocol in that URL is generated on a per-browser-instance basis, so you can't guess it ahead of time. You have to call `getURL` to generate it.

There are two important things that [the API documentation](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/web_accessible_resources) doesn't specify, which took me some time to figure out:

- **`browser.runtime.getURL` is only defined inside the extension's execution contexts.** So the content.js and background.js files are fine, but the webpage itself is not. If you want to use it in the webpage context, you'll need to generate the URL in the extension somewhere and communicate it via the `window.postMessage` event trigger.

- **Any values used in the `web_accessible_resources` parameter must be nested under a subdirectory.** If you try to call a top-level file (such as `getURL("foo.png")`), the URL will generate fine, but the file itself won't be loadable. The browser will simply report it as inaccessible.

Hopefully this saves somebody else some time.
