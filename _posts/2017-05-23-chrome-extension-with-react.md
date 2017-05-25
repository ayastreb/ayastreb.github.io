---
layout: post
title: Developing a Chrome extension with React
---

I wanted to add a couple of new features to my [Chrome extension](https://github.com/ayastreb/bandwidth-hero) and I decided to rewrite it with [React](https://facebook.github.io/react/).

My original extension was a simple one file Vanilla JS with a couple of event listeners.
In the rewrite I wanted to use ES6 modules to split my code and also give a try to React.

And even though [browsers started to support ES6 modules natively](https://medium.com/dev-channel/es6-modules-in-chrome-canary-m60-ba588dfb8ab7),
it was still in experimental stage at the time of writing, so I needed a module bundler to assemble final extension code.

The most popular way to build a React app today is [create-react-app](https://github.com/facebookincubator/create-react-app),
however it is targeted at traditional web apps and does not work well for a Chrome extension.

### Typical Chrome Extension Structure

To make functional Chrome extension you usually need at least following elements:

- [manifest.json](https://developer.chrome.com/extensions/manifest) file, which describes your extension
- [browser](https://developer.chrome.com/extensions/browserAction) or [page](https://developer.chrome.com/extensions/pageAction) action file to implement extension's popup for example
- [background page](https://developer.chrome.com/extensions/background_pages) file to implement a background process, which intercepts image loading requests in my case

So our build should be able to produce at least those 3 files separately and `create-react-app` always builds single JS bundle file.
It uses [Webpack](https://webpack.github.io/) under the hood and since I also wanted to get more familiar with Webpack,
I've decided to set it up from scratch by myself.

Webpack is a module bundler which handles all your app assets &mdash; including styles and images.
So if you need to use an image somewhere, you need to include it in the JS file which uses it like this:

{% highlight javascript %}
import logo from './assets/logo.png'

export default () => <img src={logo} />
{% endhighlight %}

### Basic Webpack Config
I've created a basic Webpack setup which builds complete extension into `./dist` folder from `./src`.

In `./src/popup.js` I include React and render my popup app from components.

In `./src/background.js` I include all background process dependencies and add required event listeners.

`./src/index.html` is a simple HTLM template which Webpack will use to build the popup file.

I had to copy `manifest.json` and icons with `CopyWebpackPlugin`, which is not true Webpack way,
but I couldn't find a plugin which will generate Chrome Extension manifest file correctly.
I should probably write that plugin myself 😀
<script src="https://gist.github.com/ayastreb/8f094c7ea17eb36cb1e6b5b9db9042c0.js"></script>

### Development Build
Webpack has built-in file watch mode, so if you run `webpack --watch` it will build the code into `./dist`
and re-build every time you change a file in `./src` which is very useful during development.
You can add `./dist` folder as unpacked Chrome extension so all your changes will be built by Webpack
and picked up by Chrome automatically!

### Production Build
When you're ready to publish your extension to Chrome Webstore you need to run `webpack -p`,
which will run a production build, minifying JS and CSS. You can than compress your `./dist` folder
and upload that file in Webstore Developer Dashboard.

### Conclusion
Overall I'm satisfied with my experience building a Chrome extension with React and Webpack.
Only the copying of manifest and icon files feels a bit like a hack, but that can be solved with a plugin 😬

You can check my extension code at [GitHub](https://github.com/ayastreb/bandwidth-hero) or in [Chrome Webstore](https://chrome.google.com/webstore/detail/bandwidth-hero/mmhippoadkhcflebgghophicgldbahdb?hl=en-US)!
