---
title: PHP Buildpack
owner: Buildpacks
menu:
  main:
    Name: PHP Buildpack
    identifier: buildpacks/php
    parent: buildpacks
---



Use the PHP buildpack with PHP or HHVM runtimes.

See the following topics:

* [Composer](./gsg-php-composer.html)
* [New Relic](./gsg-php-newrelic.html)
* [Configuration](./gsg-php-config.html)
* [Deploying and Developing PHP Apps](./gsg-php-usage.html)
* [Tips for PHP Developers](./gsg-php-tips.html)

The source for the buildpack is [here](https://github.com/cloudfoundry/php-buildpack).

## Buildpack Logging and Application Logging ##

The buildpack only runs during the staging process, and therefore only logs
what is important to staging, such as what is being downloaded, what the
configuration is, and work that the buildpack does on your application.

Once staging is done, the buildpack stops logging.
Application logging is a separate concern.

Your application must write to STDOUT or STDERR for its logs to be included in the
Loggregator stream.
See [Application Logging in Cloud Foundry](../../devguide/deploy-apps/streaming-logs.html).
