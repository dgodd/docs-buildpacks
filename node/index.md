---
title: Node.js Buildpack
owner: Buildpacks
---

<strong></strong>

Use the Node.js buildpack with Node or JavaScript applications.

See the following topics:

* [Tips for Node.js Developers](./node-tips.html)
* [Environment Variables Defined by the Node Buildpack](./node-environment.html)
* [Configure Service Connections for Node](node-service-bindings.html)

The source for the buildpack is [here](https://github.com/cloudfoundry/heroku-buildpack-nodejs).

## Buildpack Logging and Application Logging ##

The buildpack only runs during the staging process, and therefore only logs
what is important to staging, such as what is being downloaded, what the
configuration is, and work that the buildpack does on your application.

Once staging is done, the buildpack stops logging.
Application logging is a separate concern.

Your application must write to STDOUT or STDERR for its logs to be included in the
Loggregator stream.
See [Application Logging in Cloud Foundry](../../devguide/deploy-apps/streaming-logs.html).
