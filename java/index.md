---
title: Java Buildpack
owner: Java
---

<strong><%= modified_date %></strong>

Use the Java buildpack with applications written in Grails, Play, Spring or any
other JVM-based language or framework.

See the following topics:

* [Getting Started Deploying Grails Apps](./gsg-grails.html)
* [Getting Started Deploying Ratpack Apps](./gsg-ratpack.html)
* [Getting Started Deploying Spring Apps](./gsg-spring.html)
* [Tips for Java Developers](./java-tips.html)
* [Configure Service Connections for Grails](./grails-service-bindings.html)
* [Configure Service Connections for Play](./play-service-bindings.html)
* [Configure Service Connections for Spring](./spring-service-bindings.html)
* [Cloud Foundry Eclipse Plugin](./sts.html)
* [Cloud Foundry Java Client Library](./java-client.html)
* [Build Tool Integration](./build-tool-int.html)

The source for the buildpack is [here](https://github.com/cloudfoundry/java-buildpack).

## Buildpack Logging and Application Logging ##

The buildpack only runs during the staging process, and therefore only logs
what is important to staging, such as what is being downloaded, what the
configuration is, and work that the buildpack does on your application.

The Java buildpack source documentation states:

* The Java buildpack logs all messages, regardless of severity to
`<app dir>/.java-buildpack.log`.
	It also logs messages to `$stderr`, filtered by a configured severity.

* If the buildpack fails with an exception, the exception message is logged with
a log level of `ERROR` whereas the exception stack trace is logged with a log
level of `DEBUG` to prevent users from seeing stack traces by default.

Once staging is done, the buildpack stops logging.
Application logging is a separate concern.

Your application must write to STDOUT or STDERR for its logs to be included in
the Loggregator stream.
See [Application Logging in Cloud Foundry](../../devguide/deploy-apps/streaming-logs.html).









