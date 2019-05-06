---
author: "dennis"
date: "2019-05-04T13:16:08-07:00"
draft: false
title: "Understanding npm package upgrades"
tags: ["npm"]
image: "/images/content/2019/tree-dependencies.jpg"
share: true        # set false to share buttons
---

I maintain a small task management app called [Attainment](https://github.com/dencold/attainment-web) that I used to teach myself more about front-end javascript development. It uses the [Vue.js](https://vuejs.org/) framework and I use npm to manage my upstream dependencies.

I don't get much consistent time to hack on it, so it's usually several months in between development cycles. When I come back to the project, I've invariably found myself asking "what's changed and what should I update?" when it comes to my dependencies. I've frequently found myself wondering what the difference is between `npm upgrade` and `npm update` and how do I figure out _what_ dependencies are out of date in my web projects. This post is my attempt to learn the best ways to manage this and hopefully it will help others as well.

## What's the status?

My first question is "how do I know where I stand". I'd like to get a listing of things that are currently out of date. The command to do that is `npm outdated`. Here's what it looks like against Attainment after a long period of neglect:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master
$ npm outdated
Package                 Current  Wanted         Latest  Location
@vue/cli-plugin-babel     3.0.4   3.7.0          3.7.0  attainment-vue
@vue/cli-plugin-eslint    3.0.4   3.7.0          3.7.0  attainment-vue
@vue/cli-service          3.0.4   3.7.0          3.7.0  attainment-vue
bootstrap                 3.3.7   3.4.1          4.3.1  attainment-vue
chartist                 0.10.1  0.10.1         0.11.0  attainment-vue
firebase                  5.5.2  5.11.1         5.11.1  attainment-vue
fuse.js                   3.2.1   3.4.4          3.4.4  attainment-vue
moment                   2.22.2  2.24.0         2.24.0  attainment-vue
node-sass                 4.9.3  4.12.0         4.12.0  attainment-vue
vue                      2.5.17  2.6.10         2.6.10  attainment-vue
vue-datetime              0.7.1   0.7.1  1.0.0-beta.10  attainment-vue
vue-fuse                  1.5.2   1.5.2          2.0.2  attainment-vue
vue-js-modal             1.3.26  1.3.31         1.3.31  attainment-vue
vue-moment                3.2.0   3.2.0          4.0.0  attainment-vue
vue-router                3.0.1   3.0.6          3.0.6  attainment-vue
vue-template-compiler    2.5.17  2.6.10         2.6.10  attainment-vue
vuex                      3.0.1   3.1.0          3.1.0  attainment-vue
```

This gives us the dependency in the left most column, followed by the version currently installed, the one we _want_, and the latest available version upstream. Why is it that sometimes what we want is not the latest version listed? For example look at the details for `bootstrap` from that output:

```bash
Package                 Current  Wanted         Latest  Location
bootstrap                 3.3.7   3.4.1          4.3.1  attainment-vue
```

The latest version is `4.3.1`, but we want `3.4.1`. What's up with that? Well, npm understands [semver](https://semver.org/) and we have the "dependencies" section of our `package.json` file defined as:

```json
  "dependencies": {
    "bootstrap": "^3.4.1",
    "chartist": "^0.10.1",
    "firebase": "^5.11.1",
    "fuse.js": "^3.4.4",
    "moment": "^2.24.0",
    "vue": "^2.6.10",
    "vue-clickaway": "^2.2.2",
    "vue-datetime": "^0.7.1",
    "vue-fuse": "^1.5.2",
    "vue-js-modal": "^1.3.31",
    "vue-moment": "^3.2.0",
    "vue-router": "^3.0.6",
    "vuex": "^3.1.0"
  },
```

Notice that bootstrap is configured with version `^3.4.1` this tells npm that we can accept any **patch** or **minor** version updates (e.g. anything within the 3.0.0 release) safely. So in the output of `npm outated`, above, bootstrap's latest version is `4.3.1`, but we'll only accept the latest in the `3.0.0` line, which is `3.4.1`.

## Let's update!

Now that we know _what_ is out of date, and we know that npm is (hopefully) protecting us from any breaking changes in a major version change, let's update our dependencies. We do this using `npm update`:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master
$ npm update
npm WARN deprecated joi@14.3.1: This module has moved and is now available at @hapi/joi. Please update your dependencies as this version is no longer maintained an may contain bugs and security issues.
npm WARN deprecated topo@3.0.3: This module has moved and is now available at @hapi/topo. Please update your dependencies as this version is no longer maintained an may contain bugs and security issues.
npm WARN deprecated hoek@6.1.3: This module has moved and is now available at @hapi/hoek. Please update your dependencies as this version is no longer maintained an may contain bugs and security issues.

> grpc@1.20.0 install /home/coldwd/src/github.com/dencold/attainment-web/node_modules/grpc
> node-pre-gyp install --fallback-to-build --library=static_library

node-pre-gyp WARN Using request for node-pre-gyp https download 
[grpc] Success: "/home/coldwd/src/github.com/dencold/attainment-web/node_modules/grpc/src/node/extension_binary/node-v67-linux-x64-glibc/grpc_node.node" is installed via remote

> node-sass@4.12.0 install /home/coldwd/src/github.com/dencold/attainment-web/node_modules/node-sass
> node scripts/install.js

Downloading binary from https://github.com/sass/node-sass/releases/download/v4.12.0/linux-x64-67_binding.node
Download complete..] - :
Binary saved to /home/coldwd/src/github.com/dencold/attainment-web/node_modules/node-sass/vendor/linux-x64-67/binding.node
Caching binary to /home/coldwd/.npm/node-sass/4.12.0/linux-x64-67_binding.node

> node-sass@4.12.0 postinstall /home/coldwd/src/github.com/dencold/attainment-web/node_modules/node-sass
> node scripts/build.js

Binary found at /home/coldwd/src/github.com/dencold/attainment-web/node_modules/node-sass/vendor/linux-x64-67/binding.node
Testing binary
Binary is fine
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

+ fuse.js@3.4.4
+ bootstrap@3.4.1
+ moment@2.24.0
+ @vue/cli-plugin-babel@3.7.0
+ @vue/cli-plugin-eslint@3.7.0
+ node-sass@4.12.0
+ vue-js-modal@1.3.31
+ @vue/cli-service@3.7.0
+ vuex@3.1.0
+ vue-template-compiler@2.6.10
+ vue-router@3.0.6
+ vue@2.6.10
+ firebase@5.11.1
added 151 packages from 368 contributors, removed 280 packages, updated 380 packages, moved 33 packages and audited 24396 packages in 53.893s
found 1 vulnerability (1 high)
  run `npm audit fix` to fix them, or `npm audit` for details
```

That was...more output than I expected. Let's break that down.

## Deciphering update's output

The first section of output:

```bash
npm WARN deprecated joi@14.3.1: This module has moved and is now available at @hapi/joi. Please update your dependencies as this version is no longer maintained an may contain bugs and security issues.
npm WARN deprecated topo@3.0.3: This module has moved and is now available at @hapi/topo. Please update your dependencies as this version is no longer maintained an may contain bugs and security issues.
npm WARN deprecated hoek@6.1.3: This module has moved and is now available at @hapi/hoek. Please update your dependencies as this version is no longer maintained an may contain bugs and security issues.
```

That's telling me that I need to change the `joi` dependency to `@hapi/joi`, but the `joi` doesn't appear anywhere in my `package.json` file. The answer is to use the `npm ls` command:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master
$ npm ls joi
attainment-vue@0.1.0 /home/coldwd/src/github.com/dencold/attainment-web
└─┬ @vue/cli-plugin-babel@3.7.0
  └─┬ @vue/cli-shared-utils@3.7.0
    └── joi@14.3.1 
```
So, the `@vue/cli-shared-utils` package depends on `joi`, and the `@vue/cli-shared-utils` itself is a dependency of `@vue/cli-plugin-babel`. We can see this captured in the `package-lock.json` file:

```json
    "@vue/cli-shared-utils": {
      "version": "3.7.0",
      "resolved": "https://registry.npmjs.org/@vue/cli-shared-utils/-/cli-shared-utils-3.7.0.tgz",
      "integrity": "sha512-+LPDAQ1CE3ci1ADOvNqJMPdqyxgJxOq5HUgGDSKCHwviXF6GtynfljZXiSzgWh5ueMFxJphCfeMsTZqFWwsHVg==",
      "dev": true,
      "requires": {
        "chalk": "^2.4.1",
        "execa": "^1.0.0",
        "joi": "^14.3.0",
        "launch-editor": "^2.2.1",
        "lru-cache": "^5.1.1",
        "node-ipc": "^9.1.1",
        "opn": "^5.3.0",
        "ora": "^3.4.0",
        "request": "^2.87.0",
        "request-promise-native": "^1.0.7",
        "semver": "^6.0.0",
        "string.prototype.padstart": "^3.0.0"
      },
      "dependencies": {
        "semver": {
          "version": "6.0.0",
          "resolved": "https://registry.npmjs.org/semver/-/semver-6.0.0.tgz",
          "integrity": "sha512-0UewU+9rFapKFnlbirLi3byoOuhrSsli/z/ihNnvM24vgF+8sNBiI1LZPBSH9wJKUwaUbw+s3hToDLCXkrghrQ==",
          "dev": true
        }
      }
    },
```

Buuuuuttttt, where the hell does the root package `@vue/cli-plugin-babel` come from, you may be asking yourself. It **also** wasn't listed in the `package.json`. That's because I left out the `devDependencies` section of `package.json`:

```json
  "devDependencies": {
    "@vue/cli-plugin-babel": "^3.7.0",
    "@vue/cli-plugin-eslint": "^3.7.0",
    "@vue/cli-service": "^3.7.0",
    "node-sass": "^4.12.0",
    "sass-loader": "^7.0.1",
    "vue-template-compiler": "^2.6.10"
  }
```

So, I'm getting a warning for a package that is depended by another package that _itself_ is a dependency for the original package that was actually listed as a project dependency. This is a light version of what craziness in package management commonly known as [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell) in js development.

So, now we've figured out where these warnings are coming from, what do we do about them? The answer is...nothing. According to the project maintainers, these are [safe to ignore](https://github.com/vuejs/vue-cli/issues/3925#issuecomment-488564952) and are the result of transient dependencies that they don't have control over. ¯\\\_(ツ)_/¯

The next bit of output that is _actually_ helpful is this one:

```bash
+ fuse.js@3.4.4
+ bootstrap@3.4.1
+ moment@2.24.0
+ @vue/cli-plugin-babel@3.7.0
+ @vue/cli-plugin-eslint@3.7.0
+ node-sass@4.12.0
+ vue-js-modal@1.3.31
+ @vue/cli-service@3.7.0
+ vuex@3.1.0
+ vue-template-compiler@2.6.10
+ vue-router@3.0.6
+ vue@2.6.10
+ firebase@5.11.1
```

These are acutal **updates** that were performed on our dependencies, you'll see that it matches up with what we were expecting, above. For example, that `bootstrap` dependency we made a fuss about has indeed been updated to `3.4.1`. Awesome.

## Audit checks

If you were paying close attention, you probably noticed a pretty important note at the end of the output:

```bash
found 1 vulnerability (1 high)
  run `npm audit fix` to fix them, or `npm audit` for details
```

This is important and highlights security vulnerabilities that should be taken care of ASAP. Let's do as the update recommends and run `npm audit` and see what's going on here:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master
$ npm audit
                                                                                
                       === npm audit security report ===                        
                                                                                
┌──────────────────────────────────────────────────────────────────────────────┐
│                                Manual Review                                 │
│            Some vulnerabilities require your attention to resolve            │
│                                                                              │
│         Visit https://go.npm.me/audit-guide for additional guidance          │
└──────────────────────────────────────────────────────────────────────────────┘
┌───────────────┬──────────────────────────────────────────────────────────────┐
│ High          │ Arbitrary File Overwrite                                     │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Package       │ tar                                                          │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Patched in    │ >=4.4.2                                                      │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Dependency of │ node-sass [dev]                                              │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Path          │ node-sass > node-gyp > tar                                   │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ More info     │ https://npmjs.com/advisories/803                             │
└───────────────┴──────────────────────────────────────────────────────────────┘
found 1 high severity vulnerability in 24396 scanned packages
  1 vulnerability requires manual review. See the full report for details.
```

So we have a problem with the `tar` package, specifically any version before `4.4.2`. The audit tool lets us know that `tar` is a dependency of `node-sass` which is in our devDependencies of our `package.json` file. Let's confirm this using the `npm ls` command:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master
$ npm ls tar
attainment-vue@0.1.0 /home/coldwd/src/github.com/dencold/attainment-web
├─┬ firebase@5.11.1
│ └─┬ @firebase/firestore@1.2.2
│   └─┬ grpc@1.20.0
│     └─┬ node-pre-gyp@0.12.0
│       └── tar@4.4.8 
└─┬ node-sass@4.12.0
  └─┬ node-gyp@3.8.0
    └── tar@2.2.1 
```

So we have dependencies on `tar` in two places: `firebase` which is already on a version that includes the fix, and `node-sass` which mathes what we saw in the audit results. We can also use the `npm view` command to check out what the npm registry knows about the offending package:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master
$ npm view node-sass

node-sass@4.12.0 | MIT | deps: 17 | versions: 135
Wrapper around libsass
https://github.com/sass/node-sass

keywords: css, libsass, preprocessor, sass, scss, style

bin: node-sass

dist
.tarball: https://registry.npmjs.org/node-sass/-/node-sass-4.12.0.tgz
.shasum: 0914f531932380114a30cc5fa4fa63233a25f017
.integrity: sha512-A1Iv4oN+Iel6EPv77/HddXErL2a+gZ4uBeZUy+a8O35CFYTXhgA8MgLCWBtwpGZdCvTvQ9d+bQxX/QC36GDPpQ==
.unpackedSize: 1.8 MB

dependencies:
async-foreach: ^0.1.3  glob: ^7.0.3           nan: ^2.13.2           stdout-stream: ^1.4.0  
chalk: ^1.1.1          in-publish: ^2.0.0     node-gyp: ^3.8.0       true-case-path: ^1.0.2 
cross-spawn: ^3.0.0    lodash: ^4.17.11       npmlog: ^4.0.0         
gaze: ^1.0.0           meow: ^3.7.0           request: ^2.88.0       
get-stdin: ^4.0.1      mkdirp: ^0.5.1         sass-graph: ^2.2.4     

maintainers:
- am11 <adeelbm@outlook.com>
- andrewnez <andrewnez@gmail.com>
- saperski <npm@saper.info>
- xzyfer <xzyfer@gmail.com>

dist-tags:
beta: 4.11.0    latest: 4.12.0  next: 4.8.3     

published a week ago by xzyfer <xzyfer@gmail.com>
```

From this, we can see that we are indeed on on the latest version `4.12.0`. The output also lists the project's homepage which is on [github](https://github.com/sass/node-sass), checking out it's issues, I found [issue 2625](https://github.com/sass/node-sass/issues/2625) which breaks down what the package maintainers are doing to deal with this. They are waiting for _their_ dependency, `node-gyp`, to fix it for them.

Dependency hell indeed. Ughhhh.

## Checking up

Okay, after all those segues, it might be hard to remember that we did actually run a successful update. What do things look like now? If we run `npm outdated` again, here's what the results look like:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master*
$ npm outdated
Package       Current  Wanted         Latest  Location
bootstrap       3.4.1   3.4.1          4.3.1  attainment-vue
chartist       0.10.1  0.10.1         0.11.0  attainment-vue
vue-datetime    0.7.1   0.7.1  1.0.0-beta.10  attainment-vue
vue-fuse        1.5.2   1.5.2          2.0.2  attainment-vue
vue-moment      3.2.0   3.2.0          4.0.0  attainment-vue
```

Exactly as expected, all that's left are the packages that have major updates left. Nice! Running a `git status` we see:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master*
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   package-lock.json
	modified:   package.json

no changes added to commit (use "git add" and/or "git commit -a")
```

...and diffing `package.json`:

```bash
coldwd at thoreau in ~/src/github.com/dencold/attainment-web on master*
$ git d package.json

 package.json | 26 +++++++++++++-------------
 1 file changed, 13 insertions(+), 13 deletions(-)

diff --git a/package.json b/package.json
index 6bb946c..3382fba 100644
--- a/package.json
+++ b/package.json
@@ -9,26 +9,26 @@
     "lint": "vue-cli-service lint"
   },
   "dependencies": {
-    "bootstrap": "^3.3.7",
+    "bootstrap": "^3.4.1",
     "chartist": "^0.10.1",
-    "firebase": "^5.5.2",
-    "fuse.js": "^3.2.1",
-    "moment": "^2.22.2",
-    "vue": "^2.5.17",
+    "firebase": "^5.11.1",
+    "fuse.js": "^3.4.4",
+    "moment": "^2.24.0",
+    "vue": "^2.6.10",
     "vue-clickaway": "^2.2.2",
     "vue-datetime": "^0.7.1",
     "vue-fuse": "^1.5.2",
-    "vue-js-modal": "^1.3.16",
+    "vue-js-modal": "^1.3.31",
     "vue-moment": "^3.2.0",
-    "vue-router": "^3.0.1",
-    "vuex": "^3.0.1"
+    "vue-router": "^3.0.6",
+    "vuex": "^3.1.0"
   },
   "devDependencies": {
-    "@vue/cli-plugin-babel": "^3.0.4",
-    "@vue/cli-plugin-eslint": "^3.0.4",
-    "@vue/cli-service": "^3.0.4",
-    "node-sass": "^4.9.0",
+    "@vue/cli-plugin-babel": "^3.7.0",
+    "@vue/cli-plugin-eslint": "^3.7.0",
+    "@vue/cli-service": "^3.7.0",
+    "node-sass": "^4.12.0",
     "sass-loader": "^7.0.1",
-    "vue-template-compiler": "^2.5.17"
+    "vue-template-compiler": "^2.6.10"
   }
 }
```

We can see the changes correctly accounted for.

## So...what's the difference between update and upgrade?

Nothing! `npm upgrade` is an alias for `npm update`. As seen from the help output:

```bash
$ npm -h update
npm update [-g] [<pkg>...]

aliases: up, upgrade, udpate
```

However, there is, confusingly enough, an [npm-upgrade](https://www.npmjs.com/package/npm-upgrade) which is an indpendent package that gives you an interactive mode, with changelog support.

## Helpful resources

I found these articles/posts very helpful in writing up my findings:

- https://flaviocopes.com/update-npm-dependencies/
- https://stackoverflow.com/questions/36668498/what-is-the-difference-between-npm-update-g-npm-upgrade-g-npm-install
- https://medium.com/learnwithrahul/understanding-npm-dependency-resolution-84a24180901b

_Photo by Brandon Green on Unsplash_
