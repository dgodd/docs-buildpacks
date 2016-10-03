---
title: Ruby Buildpack
owner: Buildpacks
---

<strong></strong>

Use the Ruby buildpack with Ruby, Rack, Rails or Sinatra applications.

See the following topics:

* [Getting Started Deploying Ruby Apps](./gsg-ruby.html)
* [Getting Started Deploying Ruby on Rails Apps](./gsg-ror.html)
* [Deploy a Sample Ruby on Rails App](./sample-ror.html)
* [Configure a Production Server for Ruby Apps](./ruby-prod-server.html)
* [Configure Rake Tasks for Deployed Apps](./rake-config.html)
* [Tips for Ruby Developers](./ruby-tips.html)
* [Environment Variables Defined by the Ruby Buildpack](./ruby-environment.html)
* [Configure Service Connections for Ruby](./ruby-service-bindings.html)


The source for the buildpack is [here](https://github.com/cloudfoundry/cf-buildpack-ruby).

## Buildpack Logging and Application Logging ##

The buildpack only runs during the staging process, and only logs
what is important to staging, such as what is being downloaded, what the
configuration is, and work that the buildpack does on your application.

The buildpack stops logging when the staging process finishes.
Application logging is a separate concern.

If you are deploying a Ruby, Rack, or Sinatra application, your application must write to STDOUT or STDERR to include its logs in the
Loggregator stream.
See [Application Logging in Cloud Foundry](../../devguide/deploy-apps/streaming-logs.html).

If you are deploying a Rails application, the buildpack may or may not automatically install the necessary plugin or gem for logging, depending on the Rails version of the application:

* Rails 2.x: The buildpack automatically installs the `rails_log_stdout` plugin into the application.
* Rails 3.x: The buildpack automatically installs the `rails_12factor` gem if it is not present and issues a warning message.
* Rails 4.x: The buildpack only issues a warning message that the `rails_12factor` gem is not present, but does not install the gem.

You must add the `rails_12factor` gem to your `Gemfile` to quiet the warning message.

For more information about the `rails_log_stdout` plugin, refer to the [Github README](https://github.com/ddollar/rails_log_stdout).

For more information about the `rails_12factor` gem, refer to the [Github README](https://github.com/heroku/rails_12factor).
