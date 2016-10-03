---
title: Buildpack Detection
owner: Buildpacks
---

<strong></strong>

When you push an app, Cloud Foundry determines which buildpack to use.
Each buildpack has a position in the detection priority list.
Cloud Foundry first checks whether the buildpack in position 1 is right for the
app.
If the position 1 buildpack is not appropriate, Cloud Foundry moves on to the
position 2 buildpack.
This goes on until Cloud Foundry finds the correct buildpack. Otherwise,
`cf push` fails with an `Unable to detect a supported application type` error.

Each buildpack contains a detect script.
Cloud Foundry determines whether a buildpack is appropriate for an app by
running that buildpack’s detect script against the app.

By default, the three systems buildpacks occupy the first three positions
on the detection priority list.
Since running detect scripts takes time, an administrator can decrease the
average time required to push apps by making sure that the most frequently
used buildpacks occupy the lowest positions on the list.

