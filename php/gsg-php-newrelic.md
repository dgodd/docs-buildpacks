---
title: New Relic
owner: Buildpacks
---

<strong><%= modified_date %></strong>

[New Relic](http://newrelic.com/) collects analytics about application and client-side performance.

## <a id="configuration"></a> Configuration

There are two ways to configure New Relic for the PHP buildpack.

### With a CF service

Bind a NewRelic service to the app. The buildpack will automatically detect and set up NewRelic.

   This should work as long as the VCAP_SERVICES environment variable contains a service called `newrelic`. That service has a key called `credentials` and that, in turn, has a key called `licenseKey`.

   *WARNING:* This will not work with user provided services.

### With a License Key

If you already have a New Relic account, use this method.

1. Go to the New Relic website to find your [license key](https://docs.newrelic.com/docs/accounts-partnerships/accounts/account-setup/license-key).
1. Set the value of the environment variable `NEWRELIC_LICENSE` to your NewRelic license key, either through the [manifest.yml file](../../devguide/deploy-apps/manifest.html#env-block) or with the `cf set-env` command.

For more information, see
[https://github.com/cloudfoundry/php-buildpack#supported-software](https://github.com/cloudfoundry/php-buildpack#supported-software)