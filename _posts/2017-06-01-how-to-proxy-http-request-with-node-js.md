---
layout: post
title: How to proxy HTTP request with NodeJS
---

For my data compression [Chrome extension](https://github.com/ayastreb/bandwidth-hero) I needed a proxy server,
which should download the original file, compress it and respond back to browser.<br/>
It was pretty straight forward to implement using [NodeJS](https://nodejs.org) and [Express](http://expressjs.com).
<!--more-->

To handle the whole process of conversion in memory without touching the file system,
NodeJs [Streams](https://nodejs.org/api/stream.html) come in handy.
I used [request](https://github.com/request/request) library to handle image download and
[sharp](https://github.com/lovell/sharp) library to convert downloaded image.
Both of them support stream interface and work together quite nicely:

{% highlight javascript %}
const express = require('express')
const request = require('request')
const sharp = require('sharp')

const QUALITY = 40
const app = express()

app.get('/', (req, res) => {
  const imageUrl = req.query.url
  if (!imageUrl.match(/^https?:/i)) return res.status(400).end()

  const transformer = sharp()
    .grayscale()
    .toFormat('webp', { quality: QUALITY })

  request
    .get(imageUrl)     // download original image
    .pipe(transformer) // pass it to the transformer
    .pipe(res)         // pipe converted image to HTTP response
})
{% endhighlight %}

I used transform stream provided by `sharp`, but you can easily implement your own custom [transformer](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams)
to perform any kind of changes to the response from original server before returning it.

Here's the [full implementation](https://github.com/ayastreb/bandwidth-hero-proxy) of my compression proxy.
