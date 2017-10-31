# Chrome Dev Summit Keynote

MC's monica & 

- 5th year doing the summit
- TIL Downasaur

Progressive Web Apps
- trivago
- 150% increase in engagement added to homescreen
- Service worker in webkit / edge

## Workbox

- developers.google.com/ look for Workbox
- has several caching strategies builtin

Https adoption ~63% of all traffic on Chrome

One-Tap Sign-Up
- developers.google.com/identity
- Signup on mobile device, automatically signs in back on desktop chrome. cross-platform.

## The "Holy Trinity" 

- HTML: Web Components no real update, just polymer, yadda yadda
- CSS: Grid
- JS: ECMAScript 2015 fully implemented except for 1 feature (community has decided it was a badidea)
- v8 - Chrome's javascript engine. New version released. 22% faster on Speedometer. 5% faster on top 25 websites. 40% faster on ARES6.
- v8 has support for WASM (WebAssembly). available on ALL browsers (last holdout was IE). So we now have another compilation* target 
- All documentation is moving to MDN. Centralizes across webkit, chrome, edge.

## Dev tools

- Lighthouse
- Puppeteer (headless chrome instance) - anything that chrome can do puppeteer can do. CHECK THIS OUT!!! trypuppeteer.
- Webpagetest - new version Chrome User Experience Report. try this out as well.

Keynote - underwhelmed. Very polished, not a lot of amazing announcments

## A New Bar for Modern Web Standards
### Thao Tran & Chris Wilson

Talking about Scuba Diving. Old way pretty terrible data access. Contrast with HighTide. Need to prioritize for user experience. Don't just provide a service. 

Ele.me featured as the Chinese Food & Ordering app. paint in 400ms, fully interactive in 2s. They have 260M users (the same as the entire population of the United States). They are the leader in food ordering. Built out as a PWA.

Jio (Indian Entertainment).

Still seems like the same messaging. PWA's we're big in 2G countries.

- Mercado Libre (mercadolibre.com) big app in Brazil/Latin America.
- Petlove.com.br
- Flipkart
- Starbucks PWA - able to pay via barcode. Don't need a network connection.
- m.uber.com - 3s interactive on 2G, 50kb core app
- instagram - offline sync
- ebay (exporing offline) great posterchild for problem with PWA. they aren't investing in using the tech on the main site, instead doing it on ancillary pieces that doesn't mean much.

### Asking Permissions

- Right now its an ignorable request that pops up at bottom of screen (mobile)
- Moving to a modal dialog to force Block/Allow
- dc - not sure this helps at all. We really need a better way to present context

### West Elm

- November 2016 Launch West Elm beta
- March 2017 good feedback
- May 2017 rollout to 10% traffic
- June 2017 - rebuild with webcomponents
- Rollout to 25% of traffic across ios and android
- October 2017 Pottery Barn/WilliamsSonoma/West Elm retooling to new version

## Building 
### Owen Campbell-Moore

*good talk!*

Wanted to make a cross-platform messaging app. Went from Mobile Chrome Apps -> Cordova -> Java extensions. Ended up with an app that was no longer cross-platform, and was super hacky.

Installing to home screen: launched to 100% on Android. 

New image upload dialog. Really bad before. 

- Image Chooser (all we need to do is have input tag with type="file" accept="image/*" and you'll get it for free.
- Image Capture API
- Background Sync API

One tap signup/signin. Cross browser and pretty awesome.

## Faster Talk
### Addy

- Evolving RAIL
- Switching to User-happiness (First Paint, First Meaningful, First interactive)
- Parsing javascript is expensive 
- PRPL
- 5s time to full interactive is our time budget

Good tools to check out how your app is doing:
- Calibre
- Speedcurve
- BundleSize
- beta.httparchive

State of the web on mobile

Chrome User Experience Report
- available as a Big Query result set
- requires signing up for Google Cloud Platform :(
- Lookup origin for yelp!

Take advantage of "fetch" and "preload" apis for handling slow network

Announcing two new members of the PWA family:
- Pinterest
- Tinder

dc side note - a lot of the PWA "wins" are comparing the cost of building the experience. E.g. 56MB for Mobile App on iOS vs. 250kb on PWA. Seems a bit disengenious to me.

Route-based code-splitting works! Something we might want to do at Yelp.

## Service worker / Workbox
###

Using Workbox to make ServiceWorker configuration a lot easier.

workboxSW.strategies.staleWhileRevalidate

# Chrome Dev Summit - Day 2

<ruby> tag gives superscripts!
<ruby> blah blah blah <rt> howdy </rt> </ruby>

## Tooling
### Paul Irish / Eric Bidelman

#### DevTools changes

- Local caching for dev tool changes (overrides)
- Performance Monitor - live realtime views of CPU/Memory/etc.
- New Filter Sidebar on the the Console
- Put "await" in front of any async call that returns a promise, have it resolve for you

#### Headless Chrome & Puppeteer

What is a headless browser? Well there's no UI, invoked from cli. 

- --remote-debugging-port=9222
- Chrome.launch({port: 92222, chromeFlags: ['--headless']
- Selenium, PhantomJS, etc.
- Testing Headless Chrome Easy: Puppeteer. It's a node.js library
- async/await, ES6 features, also supports Node 6
- Zero config, bundles latest chromium
- Chrome's reference for using devtools
- Super easy to take a screenshot (5 lines of code)
- Easy page metrics embedding
- Has debugging options
- try-puppeteer.appspot.com

## Future of performance on the web
### Sam Saccone

- Moving into single page granular assets. E.g. page1.js, page2.js, etc. etc.
- TL;DR don't do it. Chrome team tried to do this with moment.js. Unbundling caused an extra 100ms time.
- Lots of stuff were proposals...not shipped, why even tell us about it?
- Example "DOMChangeList"
-

## V8 Talk

- 22% better and 40% better on two major benchmarks since last year
- Reducing Jank --> typically due to GC
- V8 team is taking garbage collection off of the main thread
- They leave some small slivers on the main thread, but now moved off and takes advantage of multiple cores

## Real World WASM
### 

- A new capability for the web
- Compile target for languages used in native apps
- Binary format, safer than JS

## Polymer Talk
### Taylor Savage

- The javascript-industrial-complex: feedback loop that encourages short term gains
- "beware the local maximum"
- path to a new web platform feature. very slooowwww...
- spec proposal, implementation, ship, fix, wait for other browsers to catch up
- web components & polymer

## Javascript Frameworks Panel Discussion
### Moderated by Addy


- "Anything you are looking forward to" DOM Changelist! Moving more code to be executed on WebWorker and off of main thread.
- Observables - Angular uses this a lot. Mutation observer etc., move to platform. Would be way better if we have primitives.
- Moving from callbacks to promises. Being able to pass along an object.
- Changelist + Worker DOM. 
- Async append -> build up a tree of DOM and asyncronously append into the dom.
- Andrew Clark: (React) "Except for redesigning layout which we can't, but maybe we will". WTF?
- What framework would you use? 
- Frameworks...you only have about 130kb, how much of that 130kb is your framework taking up?
- Looks over at React. 
- Showing that there is a million+1 js frameworks, check out the stage presence at the panel discussion
-
