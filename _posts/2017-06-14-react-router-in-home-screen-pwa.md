---
layout: post
title: Using React Router in Home Screen PWA
---

In an app I've been working on recently I'm using [React Router](https://github.com/ReactTraining/react-router)
with HTML5 History API. The app is a [Progressive Web App](https://developers.google.com/web/progressive-web-apps/)
and it turns out that History API routing does not work when app is launched from home screen.
<!--more-->

It took me a bit of time to figure out why my app works in Chrome Mobile, but doesn't
work when launched from home screen. To work around this issue I've decided to use [hash router](https://reacttraining.com/react-router/web/api/HashRouter)
when app is launched from home screen and [history router](https://reacttraining.com/react-router/web/api/BrowserRouter) when app is running in browser.

The [recommended way](https://developers.google.com/web/updates/2015/10/display-mode)
to check if app is running in home screen is to use [media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries).

{% highlight javascript %}
if (window.matchMedia('(display-mode: standalone)').matches) {
  console.log("We are in home screen");
}
{% endhighlight %}

So in my app instead of always using `<BrowserRouter>` I now use `<Router>` and
pass history object as a prop to it.

{% highlight javascript %}
// configureHistory.js
import { createBrowserHistory, createHashHistory } from 'history'

export default function configureHistory() {
  return window.matchMedia('(display-mode: standalone)').matches
    ? createHashHistory()
    : createBrowserHistory()
}

// index.js
import { Router, Route } from 'react-router-dom'
import { configureHistory } from './configureHistory.js'

const history = configureHistory()
ReactDOM.render(
  <Router history={history}>
    ...
    <Route ... />
  </Router>,
  document.getElementById('root')
)
{% endhighlight %}

This way the app uses hash router only when launched from home screen.
