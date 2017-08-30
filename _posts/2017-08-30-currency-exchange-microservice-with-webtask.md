---
layout: post
title: Currency Exchange Microservice with Webtask.io
---

The finance data-table in [Yahoo APIs](https://developer.yahoo.com/yql/) has recently stopped working, returning error `"No definition found for Table yahoo.finance.xchange"`. I need currency exchange rates for my [money tracking app](https://github.com/ayastreb/money-tracker),
so I had to look for another provider.
<!--more-->

After googling around I found [CurrencyLayer](https://currencylayer.com/) - a service which provides
exchange rates for 168 currencies and even have a free plan. This plan is
limited with 1,000 requests per month.

I've used [webtask.io](https://webtask.io/) before in other projects and
I got an idea to write a microservice which would cache exchange rate from CurrencyLayer.
I would only need to cache USD-based rates and calculate non-USD based rates
using cross-rate.
Webtasks provide a [single JSON document storage](https://webtask.io/docs/storage)
which would be perfect for our use-case.

If we cache the exchange rate for 1 hour, it will use max. 744 requests per month  (24 hours x 31 days),
which should fit into the free plan 😉

### Design
The microservice will have a very simple interface - an HTTP GET endpoint with a single `pairs` parameter.
This parameter will containt a list of currency exchange pairs separated by comma.
Sample URL request would look like this:
{% highlight bash %}
curl -XGET https://endpoint-url/?pairs=USDEUR,EURUSD,USDJPY
{% endhighlight %}
and will respond with a list of exchange rates for given pairs:
{% highlight json %}
{
    "ok": "true",
    "rates": [
       {
         "id": "USDEUR",
         "rate": "0.834303"
       },
       {
         "id": "EURUSD",
         "rate": "1.198605"
       },
       {
         "id": "USDJPY",
         "rate": "109.678001"
       }
    ]
}
{% endhighlight %}

### Implementation
Implementing this is pretty straight forward.

A webtask is just a [JavaScript function](https://webtask.io/docs/model) with two arguments.

First, [`context`](https://webtask.io/docs/context) - gives us access to URL query parameters,
webtask [secrets](https://webtask.io/docs/editor/secrets) (to keep our API keys separated from code)
and [`storage API`](https://webtask.io/docs/storage) to cache data.

Second, `respond` - is a callback function which we should call when we're finished.
It takes an `error` and `result` arguments and converts them to JSON.

Here's the [code](https://github.com/ayastreb/money-tracker/blob/master/src/webtasks/CurrencyExchange.js) of this webtask.
{% highlight javascript %}
module.exports = function(context, respond) {
  const request = require('request')
  const BASE = 'USD'
  const pairs = context.query.pairs.toUpperCase().split(',')

  return getBaseRate()
    .then(baseRate =>
      respond(null, {
        ok: true,
        rates: pairs.map(pair => getRateForPair(baseRate, pair))
      })
    )
    .catch(error => respond(error))

  /**
   * Get exchange rate for given pair using given base exchange rate.
   * Use cross rate if pair's base is not equal to rate base (USD).
   *
   * @param {object} baseRate - dict { USD: 1, EUR: 0.834499, ... }
   * @param {string} pair - "USDEUR", "EURUSD", "EURJPY", etc
   * @return {object} dict { id: "USDEUR", rate: "0.834499" }
   */
  function getRateForPair(baseRate, pair) {
    if (pair.length != 6) {
      throw new Error(
        `Invalid pair "${pair}". Must be 6-char string, e.g. "USDEUR"`
      )
    }

    const source = pair.substr(0, 3)
    const target = pair.substr(3, 3)

    if (!baseRate[source]) throw new Error(`Unknown currency code "${source}"`)
    if (!baseRate[target]) throw new Error(`Unknown currency code "${target}"`)

    return {
      id: pair,
      rate: Number(
        source === BASE
          ? baseRate[target]
          : 1 / baseRate[source] * baseRate[target]
      ).toFixed(6)
    }
  }

  /**
   * Get exchange rate for base currency (USD).
   *
   * @return {object} dict { USD: 1, EUR: 0.834499, ... }
   */
  function getBaseRate() {
    return fetchCachedRate()
      .then(rate => rate, error => fetchLiveRate())
      .then(checkCachedRateAge)
      .then(convertRate)
  }

  /**
   * Read cached rate from webtask storage.
   *
   * @see https://webtask.io/docs/storage
   * @return {Promise}
   */
  function fetchCachedRate() {
    return new Promise((resolve, reject) => {
      context.storage.get((error, rate) => {
        if (error) return reject(error)
        if (rate === undefined) return reject()

        resolve(rate)
      })
    })
  }

  /**
   * Fetch base exchange rate from CurrencyLayer live API.
   * Fallback to cached rate if API is not available.
   *
   * @see https://currencylayer.com/documentation
   * @see https://webtask.io/docs/editor/secrets
   * @return {Promise}
   */
  function fetchLiveRate() {
    return new Promise(resolve => {
      const apiKey = context.secrets.apiKey
      request(
        {
          method: 'GET',
          uri: `http://apilayer.net/api/live?access_key=${apiKey}`,
          json: true
        },
        (error, response, body) => {
          if (error || !body.success) {
            fetchCachedRate().then(rate => resolve(rate))
          } else {
            writeCachedRate(body).then(() => resolve(body))
          }
        }
      )
    })
  }

  /**
   * Write given rate to webtask cache.
   *
   * @see https://webtask.io/docs/storage
   * @param {object} rate
   * @return {Promise}
   */
  function writeCachedRate(rate) {
    return new Promise((resolve, reject) => {
      context.storage.set(
        rate,
        { force: 1 },
        error => (error ? reject(error) : resolve())
      )
    })
  }

  /**
   * Validate cached rate expiry date.
   *
   * @param {object} rate
   */
  function checkCachedRateAge(rate) {
    const expiryDate = Math.floor(Date.now() / 1000) - 3600
    return rate.timestamp < expiryDate ? fetchLiveRate() : rate
  }

  /**
   * Convert response from API service to internal rate object.
   *
   * @param {object} dict { ..., quotes: { USDUSD: 1, USDEUR: 0.834499, ... } }
   * @return {object} dict { USD: 1, EUR: 0.834499, ... }
   */
  function convertRate(rate) {
    return Object.keys(rate.quotes).reduce((acc, pair) => {
      const code = pair.substr(3, 3)
      acc[code] = rate.quotes[pair]
      return acc
    }, {})
  }
}
{% endhighlight %}

You can check my webtask running [here](https://wt-e9c9a7a436fcd9273a7f8890849dae65-0.run.webtask.io/currency-exchange?pairs=USDEUR,EURUSD,USDJPY).

If you plan to use this approach in your own projects -
I suggest getting your own [CurrencyLayer](https://currencylayer.com) account and [running](https://webtask.io/docs/101)
your own webtask.

