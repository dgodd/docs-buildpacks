---
title: Buildpacks for the Enterprise
owner: Buildpacks
---

<strong></strong>

## <a id='intro'></a>Introduction to Buildpack Dependencies ##

Buildpacks often need access to runtime binaries for the platform that they target.

For example, depending on the source that it builds from, the [Ruby buildpack for Heroku](https://github.com/heroku/heroku-buildpack-ruby) requires the following binaries:

  * A Ruby interpreter
  * Libyaml
  * Bundler
  * Node

This Ruby buildpack uses public accessible Amazon S3 hosting for these dependencies.

## <a id='online'></a>Online Cloud Foundry Deployments ##

Online deployments of Cloud Foundry, such as [Pivotal Web Services](https://run.pivotal.io/), generally work well with third-party and Heroku buildpacks without modification.
You can specify the use of these buildpacks with the `-b` option to the Cloud
Foundry CLI.

When building droplets, the [Droplet Execution Agent (DEA)](../concepts/architecture/execution-agent.html)
executes the buildpack within a container whose network can reach the Internet
entirely unimpeded.

Many Pivotal customers do not allow the DEA, or any part of their Cloud Foundry
deployment, unimpeded access to the Internet.
Security and Change Management concerns impact how an enterprise or smaller business business decides to architect their network.

To satisfy these customers, Pivotal forks existing buildpacks, or creates customized ones, which support the loading of dependencies from behind a firewall. When building your buildpack, you should consider these mechanisms, especially if you want enterprise companies to adopt your platform.

For a breakdown of offline modes and how to implement them, see the [Offline Modes](./offline_modes.md) topic.

You can find buildpacks specific to Cloud Foundry-specific in the [Cloud Foundry wiki](https://github.com/cloudfoundry-community/cf-docs-contrib/wiki/Buildpacks.