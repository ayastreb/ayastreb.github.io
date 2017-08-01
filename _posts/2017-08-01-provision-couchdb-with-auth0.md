---
layout: post
title: Provision CouchDB users with Auth0
---

In my [personal finances tracking app](https://github.com/ayastreb/money-tracker) I use
[PouchDB](https://pouchdb.com/) to store data locally in the browser
and I wanted to sync the data to the server-side CouchDB, so that users can access their data from multiple devices.
<!--more-->

Since all data in the app is private to user and users don't share any data,
the "Database per User" pattern felt natural to use. In this pattern we create new database and user in CouchDB
for each new user in our app. We then use this personal CouchDB to sync data with PouchDB in the browser.

### Auth0 for authentication

I've decided to use [Auth0 Passwordless](https://auth0.com/passwordless) for users authentication because with them
I don't need to spend much time implementing whole auth workflow. Auth0 also makes it easy to [extend](https://auth0.com/docs/topics/extensibility)
 authentication workflow with custom rules. [Rule](https://auth0.com/docs/rules/current) is basically a JavaScript function which runs on certain events during authentication.

### Cloudant &mdash; CouchDB as a Service

At first I found a [great article](https://www.bennadel.com/blog/3208-provisioning-cloudant-couchdb-databases-from-auth0-for-a-database-per-user-architecture-in-angular-2-4-1.htm)
 by Ben Nadel, which shows how to create user and database in Cloudant after authentication in Auth0.
 [Cloudant](https://www.ibm.com/analytics/us/en/technology/cloud-data-services/cloudant/) is now owned by IBM and offered as part of their IBM Bluemix cloud platform.
 They provide perpetually free Lite plan which is limited by throughput (e.g. 20 lookups,10 writes and 5 queries per second).

 I've setup a test account with Cloudant and tried to sync data with PouchDB in the browser,
 but I was constantly getting time-out errors from Cloudant after just 2 connected clients doing live-replication.

### Self-hosted CouchDB

The cheapest Cloudant plan after Lite costs $80 per month which I didn't want to spent on a side project.
After browsing around I've decided to host my own instance of CouchDB server at [Google Cloud](https://cloud.google.com/).

They provide one year free trial and a free tiny instance after the trial ends.
It's also easy to deploy basic CouchDB server with [Bitnami Launcher](https://console.cloud.google.com/launcher/details/bitnami-launchpad/couchdb).
You will need to setup networking and load-balancing to make CouchDB server publicly available via HTTP, I will describe that process in more detail in a separate blog post.

### Provision Workflow

Auth0 provides [metadata](https://auth0.com/docs/metadata) for each user, which we can manipulate from our custom rules.
We can also access this user metadata from our app after successful authentication and use it for data synchronization.
High level provision workflow would look something like this:
- when user is authenticated, check if they have CouchDB credentials in their metadata
- if no CouchDB credentials stored in metadata - provision new user and databases in CouchDB server
- store CouchDB credentials in metadata

We need to create admin user in our CouchDB server and provide this admin credentials to our rule code.
There is a simple key-value configuration interface in Auth0 which allows you to do just that - set our
CouchDB host, admin user and password from Auth0's web interface and access it in rule via `configuration` global object.

### Code Sample

Here's the [full code](https://github.com/ayastreb/money-tracker/blob/master/src/webtasks/CouchDbProvision.js) of the rule I use to provision CouchDB users and databases with Auth0.
I actually create multiple databases for each user, but that is not important.
I use [request](https://github.com/request/request) to perform plain HTTP requests to CouchDB.
I also use [uuid](https://github.com/kelektiv/node-uuid) to generate random username and password.

As a first step I log in as admin to our CouchDB server and store auth cookie in a variable,
which is later used for all HTTP requests. After that I create new user in CouchDB, new databases and set the newly created user as the only one having access to their databases.
After that I update user's metadata with newly created CouchDB credentials.

{% highlight javascript %}
function CouchDbProvision(user, context, callback) {
  const request = require('request')
  const uuidv4 = require('uuid').v4
  const databases = ['accounts', 'transactions', 'tags', 'settings']
  var cookie

  user.app_metadata = user.app_metadata || {}
  user.app_metadata.couchDB = user.app_metadata.couchDB || {}

  loginAsAdmin()
    .then(provisionUser)
    .then(() => Promise.all(databases.map(provisionDatabase)))
    .then(() => Promise.all(databases.map(provisionSecurity)))
    .then(() => auth0.users.updateAppMetadata(user.user_id, user.app_metadata))
    .then(() => callback(null, user, context))
    .catch(err => callback(err))

  function loginAsAdmin() {
    return new Promise((resolve, reject) => {
      request(
        {
          method: 'POST',
          uri: `${configuration['CouchHost']}/_session`,
          json: true,
          body: {
            name: configuration['CouchAdminUser'],
            password: configuration['CouchAdminPass']
          }
        },
        (err, response) => {
          if (err) return reject(err)

          cookie = response.headers['set-cookie']
          resolve()
        }
      )
    })
  }

  function provisionUser() {
    return new Promise((resolve, reject) => {
      if (user.app_metadata.couchDB.username) return resolve()

      const name = uuidv4()
      const password = uuidv4()
      request(
        {
          method: 'PUT',
          headers: { cookie },
          uri: `${configuration['CouchHost']}/_users/org.couchdb.user:${name}`,
          json: true,
          body: { name, password, roles: [], type: 'user' }
        },
        (err, response, body) => {
          if (err) return reject(err)
          if (!body.ok) return reject('Could not create user')

          user.app_metadata.couchDB.username = name
          user.app_metadata.couchDB.password = password
          resolve()
        }
      )
    })
  }

  function provisionDatabase(name) {
    return new Promise((resolve, reject) => {
      if (user.app_metadata.couchDB[name]) return resolve()

      request(
        {
          method: 'PUT',
          headers: { cookie },
          uri: databaseUri(name),
          json: true
        },
        (err, response, body) => {
          if (err) return reject(err)
          if (!body.ok) return reject('Could not create database')

          resolve()
        }
      )
    })
  }

  function provisionSecurity(name) {
    return new Promise((resolve, reject) => {
      if (user.app_metadata.couchDB[name]) return resolve()

      request(
        {
          method: 'PUT',
          headers: { cookie },
          uri: `${databaseUri(name)}/_security`,
          json: true,
          body: {
            admins: { names: [], roles: [] },
            members: { names: [user.app_metadata.couchDB.username], roles: [] }
          }
        },
        (err, response, body) => {
          if (err) return reject(err)
          if (!body.ok) return reject('Could not set database security')

          user.app_metadata.couchDB[name] = databaseUri(name)
          resolve()
        }
      )
    })
  }

  function databaseUri(name) {
    return `${configuration['CouchHost']}/${name}_${user.app_metadata.couchDB.username}`
  }
}
{% endhighlight %}
