---
title: Configure Service Connections for Ruby
owner: Buildpacks
---

<strong><%= modified_date %></strong>

_This page assumes that you are using cf CLI v6._

After you create a service instance and bind it to an application, you must
configure the application to connect to the service.

## <a id='cf-app-utils'></a>Query VCAP\_SERVICES with cf-app-utils ##

The `cf-apps-utils` gem allows your application to search for credentials from
the `VCAP_SERVICES` environment variable by name, tag, or label.

* [cf-app-utils-ruby](https://github.com/cloudfoundry/cf-app-utils-ruby)

## <a id='vcap-services-defines-database-url'></a>VCAP\_SERVICES defines DATABASE\_URL ##

At runtime, the Ruby buildpack creates a `DATABASE_URL` environment variable
for every Ruby application based on the
[VCAP_SERVICES](../../devguide/deploy-apps/environment-variable.html#VCAP-SERVICES) environment variable.

Example VCAP_SERVICES:

    VCAP_SERVICES =
    {
      "elephantsql": [
        {
          "name": "elephantsql-c6c60",
          "label": "elephantsql",
          "credentials": {
            "uri": "postgres://exampleuser:examplepass@babar.elephantsql.com:5432/exampledb"
          }
        }
      ]
    }

Based on this `VCAP_SERVICES`, the Ruby buildpack creates the following
`DATABASE_URL` environment variable:

    DATABASE_URL = postgres://exampleuser:examplepass@babar.elephantsql.com:5432/exampledb

The Ruby buildpack uses the structure of the `VCAP_SERVICES` environment
variable to populate `DATABASE_URL`.
Any service containing a JSON object with the following form will be recognized
by Cloud Foundry as a candidate for `DATABASE_URL`:

    {
      "some-service": [
        {
          "credentials": {
            "uri": "<some database URL>"
          }
        }
      ]
    }

Cloud Foundry uses the first candidate found to populate `DATABASE_URL`.

## <a id='rails-applications-have-autoconfigured-database-yml'></a>Older Rails Applications Have Auto-Configured database.yml ##

On Rails versions 4 or before, the Ruby buildpack replaces your `database.yml` with one based on the `DATABASE_URL` variable.

<p class='note'><strong>Note</strong>: On Rails versions 4 or before, Ruby buildpack ignores the contents of any <code>database.yml</code> that you provide and overwrites it during staging.</p>

## <a id='configuring-non-rails-applications'></a>Configuring Non-Rails Applications ##

Non-Rails applications can also access the `DATABASE_URL` variable.

If you have more than one service with credentials, only the first will be
populated into `DATABASE_URL`.
To access other credentials, you can inspect the `VCAP_SERVICES` environment
variable.

~~~ruby
vcap_services = JSON.parse(ENV['VCAP_SERVICES'])
~~~

Use the hash key for the service to obtain the connection credentials from
`VCAP_SERVICES`.

- For services that use the <%= vars.api_v2_format %>, the hash key is the name
  of the service.

- For services that use the <%= vars.api_v1_format %>, the hash key is formed
  by combining the service provider and version, in the format PROVIDER-VERSION.

  For example, the service provider "p-mysql" with version "n/a" forms the hash
  key `p-mysql-n/a`.

## <a id='migrate'></a>Seed or Migrate Database ##

Before you can use your database the first time, you must create and populate
or migrate it.
For more information, see [Migrating a Database in Cloud Foundry](../../devguide/services/migrate-db.html).

## <a id='troubleshooting'></a>Troubleshooting ##

To aid in troubleshooting issues connecting to your service, you can examine
the environment variables and log messages Cloud Foundry records for your
application.

### <a id='view-env'></a>View Environment Variables ###

Use the `cf env` command to view the Cloud Foundry environment variables for
your application.
`cf env` displays the following environment variables:

* The `VCAP_SERVICES` variables existing in the container environment
* The user-provided variables set using the `cf set-env` command

<pre class="terminal">
  $ cf env my-app
  Getting env variables for app my-app in org My-Org / space development as admin...
  OK

  System-Provided:
  {
    "VCAP_SERVICES": {
      "p-mysql-n/a": [
        {
          "credentials": {
      	    "uri":"postgres://lrra:e6B-X@p-mysqlprovider.example.com:5432/lraa
          },
          "label": "p-mysql-n/a",
          "name": "p-mysql",
          "syslog_drain_url": "",
          "tags": ["postgres","postgresql","relational"]
        }
      ]
    }
  }

  User-Provided:
  my-env-var: 100
  my-drain: http://drain.example.com
</pre>

### <a id='view-logs'></a>View Logs ###

Use the `cf logs` command to view the Cloud Foundry log messages for your
application.
You can direct current logging to standard output, or you can dump the most
recent logs to standard output.

Run `cf logs APPNAME` to direct current logging to standard output:

<pre class="terminal">
  $ cf logs my-app
  Connected, tailing logs for app my-app in org My-Org / space development as admin...
  1:27:19.72 [App/0]  OUT [CONTAINER]  org.apache.coyote.http11.Http11Protocol  INFO  Starting ProtocolHandler ["http-bio-61013"]
  1:27:19.77 [App/0]  OUT [CONTAINER]   org.apache.catalina.startup.Catalina     INFO  Server startup in 10427 ms
</pre>

Run `cf logs APPNAME --recent` to dump the most recent logs to standard output:
<pre class="terminal">
  $ cf logs my-app --recent
  Connected, dumping recent logs for app my-app in org My-Org / space development as admin...
  1:27:15.93 [App/0]  OUT 15,935  INFO EmbeddedDatabaseFactory:124 - Creating embedded database 'SkyNet'
  1:27:16.31 [App/0]  OUT 16,318  INFO LocalEntityManagerFactory:287 - Building TM container EntityManagerFactory for unit 'default'
  1:27:16.50 [App/0]  OUT 16,505  INFO Version:37 - HCANN001: Hibernate Commons Annotations {4.0.1.Final}
  1:27:16.51 [App/0]  OUT 16,517  INFO Version:41 - HHH412: Hibernate Core {4.1.9.Final}
  1:27:16.95 [App/0]  OUT 16,957  INFO SkyNet-Online:73 - HHH268: Transaction strategy: org.hibernate.internal.TransactionFactory
  1:27:16.96 [App/0]  OUT 16,963  INFO InintiateTerminatorT1000Deployment:48 - HHH000397: Using TranslatorFactory
  1:27:17.02 [App/0]  OUT 17,026  INFO Version:27 - HV001: Hibernate Validator 4.3.0.Final
</pre>

If you encounter the error, "A fatal error has occurred. Please see the Bundler
troubleshooting documentation," update your version of bundler and run `bundle install`.

<pre class="terminal">
  $ gem update bundler
  $ gem update --system
  $ bundle install
</pre>
