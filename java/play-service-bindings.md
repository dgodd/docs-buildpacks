---
title: Configure Service Connections for Play Framework
owner: Java
menu:
  main:
    Name: Configure Service Connections for Play Framework
    identifier: buildpacks/java/play-service-bindings
    parent: buildpacks/java
---



Cloud Foundry provides support for connecting a Play Framework application to services such as MySQL and Postgres. In many cases, a Play Framework application running on Cloud Foundry can automatically detect and configure connections to services.


## <a id='auto'></a>Auto-Configuration ##
By default, Cloud Foundry detects service connections in a Play Framework application and configures them to use the credentials provided in the Cloud Foundry environment. Note that auto-configuration happens only if there is a single service of either of the supported types&#8212;MySQL or Postgres.