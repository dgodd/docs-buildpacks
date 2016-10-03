---
title: Supported Binary Dependencies
owner: Buildpacks
---

<strong><%= modified_date %></strong>

Each buildpack only supports the stable patches for each dependency listed in the buildpack's `manifest.yml` and also in its GitHub releases page.
For example, see the [php-buildpack releases page](https://github.com/cloudfoundry/php-buildpack/releases).

If you try to use a binary that is not currently supported, staging your app fails with the following error message:

```
Could not get translated url, exited with: DEPENDENCY_MISSING_IN_MANIFEST:
...
 !
 !     exit
 !
Staging failed: Buildpack compilation step failed
```
