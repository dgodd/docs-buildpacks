---
title: Tips for Ruby Developers
owner: Buildpacks
---

<strong></strong>

_This page assumes you are using cf CLI v6._

This page has information specific to deploying Rack, Rails, or Sinatra
applications.

## <a id='bundler'></a>Application Bundling ##

You must run <a href="http://gembundler.com/">Bundler</a> to create a `Gemfile`
and a `Gemfile.lock`.
These files must be in your application before you push to Cloud Foundry.

## <a id='config'></a>Rack Config File ##

For Rack and Sinatra, you must have a `config.ru` file. For example:

~~~ruby
require './hello_world'
run HelloWorld.new
~~~

## <a id='precompile'></a>Asset Precompilation ##

Cloud Foundry supports the Rails asset pipeline.
If you do not precompile assets before deploying your application, Cloud Foundry
will precompile them when staging the application.
Precompiling before deploying reduces the time it takes to stage an application.

Use the following command to precompile assets before deployment:

<pre class="terminal">
rake assets:precompile
</pre>

Note that the Rake precompile task reinitializes the Rails application.
This could pose a problem if initialization requires service connections or
environment checks that are unavailable during staging.
To prevent reinitialization during precompilation, add the following line to
`application.rb`:

~~~ruby
config.assets.initialize_on_precompile = false
~~~

If the `assets:precompile` task fails, Cloud Foundry uses live compilation mode,
the alternative to asset precompilation.
In this mode, assets are compiled when they are loaded for the first time.
You can force live compilation by adding the following line to `application.rb`.

~~~ruby
Rails.application.config.assets.compile = true
~~~

## <a id='rake'></a>Running Rake Tasks ##

Cloud Foundry does not provide a mechanism for running a Rake task on a deployed
application.
If you need to run a Rake task that must be performed in the Cloud Foundry
environment, rather than locally before deploying or redeploying, you can
configure the command that Cloud Foundry uses to start the application to invoke
the Rake task.

An application's start command is configured in the application's manifest file,
`manifest.yml`, using the `command` attribute.

If you have previously deployed the application, the application manifest should
already exist.
There are two ways to create a manifest.
You can manually create the file and save it in the application's root directory
before you deploy the application for the first time.
If you do not manually create the manifest file, the cf CLI will prompt you to
supply deployment settings when you first push the application, and will create
and save the manifest file for you, with the settings you specified
interactively.
For more information about application manifests, and supported attributes, see
[Deploying with Application Manifests](../../devguide/deploy-apps/manifest.html).

### <a id='migrate-ruby-db'></a>Example: Invoking a Rake database migration task at application startup ###

The following is an example of the "migrate frequently" method described in the
[Migrating a Database in Cloud Foundry]
(../../devguide/services/migrate-db.html#frequent-migration) topic.

1. Create a Rake task to limit an idempotent command to the first instance of a deployed application:

    ~~~
    namespace :cf do
      desc "Only run on the first application instance"
      task :on_first_instance do
        instance_index = JSON.parse(ENV["VCAP_APPLICATION"])["instance_index"] rescue nil
        exit(0) unless instance_index == 0
      end
    end
    ~~~

2. Add the task to the `manifest.yml` file, referencing the idempotent command `rake db:migrate` with the `command` attribute.

    ~~~
   ---
    applications:
    - name: my-rails-app
      command: bundle exec rake cf:on_first_instance db:migrate && bundle exec rails s -p $PORT -e $RAILS_ENV
    ~~~

3. Update the application using `cf push`.

## <a id='workers'></a>Rails 3 Worker Tasks ##

This section shows you how to create and deploy an example Rails application
that uses a worker library to defer a task that a separate application executes.

The guide also describes how to scale the resources available to the worker
application.

<p class="note"><strong>Note</strong>: Most worker tasks do not serve external requests. Use the <code>--no-route</code> flag with the <code>cf push</code> command, or <code>no-route: true</code> in the application manifest, to suppress route creation and remove existing routes.</p>

### <a id='worker-libs'></a>Choose a Worker Task Library ###

You must choose a worker task library.
The table below summarizes the three main libraries available for Ruby / Rails:

<table border="1" class="nice">
  <tr>
    <th>Library</th>
    <th>Description</th>
  </tr>
  <tr>
 <td><a href="https://github.com/collectiveidea/delayed_job">Delayed::Job</a></td>
    <td>A direct extraction from <a href="http://www.shopify.com/">Shopify</a> where the job table is responsible for a multitude of core tasks.</td>
  </tr>
  <tr>
    <td><a href="https://github.com/defunkt/resque">Resque</a></td>
    <td>A Redis-backed library for creating background jobs, placing those jobs on multiple queues, and processing them later.</td>
  </tr>
  <tr>
    <td><a href="https://github.com/mperham/sidekiq">Sidekiq</a></td>
    <td>Uses threads to handle many messages at the same time in the same process. It does not require Rails but will integrate tightly with Rails 3 to make background message processing dead simple. This library is also Redis-backed and is actually somewhat compatible with Resque messaging.</td>
  </tr>
</table>

For other alternatives, see
[https://www.ruby-toolbox.com/categories/Background_Jobs](https://www.ruby-toolbox.com/categories/Background_Jobs).

### <a id='example-app'></a>Create an Example Application ###

For the purposes of the example application, we will use Sidekiq.

First, create a Rails application with an arbitrary model called "Things":

<pre class="terminal">
$ rails create rails-sidekiq
$ cd rails-sidekiq
$ rails g model Thing title:string description:string
</pre>

Add `sidekiq` and `uuidtools` to the Gemfile:

~~~ruby
source 'https://rubygems.org'

gem 'rails', '3.2.9'
gem 'mysql2'

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

gem 'jquery-rails'
gem 'sidekiq'
gem 'uuidtools'
~~~

Install the bundle.

<pre class="terminal">
$ bundle install
</pre>

Create a worker (in app/workers) for Sidekiq to carry out its tasks:

<pre class="terminal">
$ touch app/workers/thing_worker.rb
</pre>

~~~ruby
class ThingWorker

  include Sidekiq::Worker

  def perform(count)

    count.times do

      thing_uuid = UUIDTools::UUID.random_create.to_s
      Thing.create :title =>"New Thing (#{thing_uuid})", :description =>
"Description for thing #{thing_uuid}"
    end

  end

end
~~~

This worker will create n number of things, where n is the value passed to the
worker.

Create a controller for "Things":

<pre class="terminal">
$ rails g controller Thing
</pre>

~~~ruby
class ThingController < ApplicationController

  def new
    ThingWorker.perform_async(2)
    redirect_to '/thing'
  end

  def index
    @things = Thing.all
  end

end
~~~

Add a view to inspect our collection of "Things":

<pre class="terminal">
$ mkdir app/views/things
$ touch app/views/things/index.html.erb
</pre>

```
<%= @things.inspect %>
```


#### <a id='deploy'></a>Deploy the Application ####

This application needs to be deployed twice for it to work, once as a Rails web
application and once as a standalone Ruby application.
The easiest way to do this is to keep separate Cloud Foundry manifests for each
application type:

Web Manifest: Save this as `web-manifest.yml`:

~~~yaml
---
applications:
- name: sidekiq
  memory: 256M
  instances: 1
  host: sidekiq
  domain: ${target-base}
  path: .
  services:
  - sidekiq-mysql:
  - sidekiq-redis:
~~~

Worker Manifest: Save this as `worker-manifest.yml`:

~~~yaml
---
applications:
- name: sidekiq-worker
  memory: 256M
  instances: 1
  path: .
  command: bundle exec sidekiq
  no-route: true
  services:
  - sidekiq-redis:
  - sidekiq-mysql:
~~~

Since the url "sidekiq.cloudfoundry.com" is probably already taken, change it
in `web-manifest.yml` first, then push the application with both manifest
files:

<pre class="terminal">
$ cf push -f web-manifest.yml
$ cf push -f worker-manifest.yml
</pre>

If the cf CLI asks for a URL for the worker application, select "none".

### <a id='test'></a>Test the Application ###

Test the application by visiting the new action on the "Thing" controller at
the assigned url.
In this example, the URL would be `http://sidekiq.cloudfoundry.com/thing/new`.

This will create a new Sidekiq job which will be queued in Redis, then picked
up by the worker application.
The browser is then redirected to `/thing` which will show the collection of
"Things".

### <a id='scale'></a>Scale Workers ###

Use the `cf scale` command to change the number of Sidekiq workers.

Example:

<pre class="terminal">
$ cf scale sidekiq-worker -i 2
</pre>

## <a id='rails-4'></a>Use rails\_serv\_static\_assets on Rails 4 ##

By default Rails 4 returns a 404 if an asset is not handled via an external
proxy such as Nginx.
The `rails_serve_static_assets` gem enables your Rails server to deliver
static assets directly, instead of returning a 404.
You can use this capability to populate an edge cache CDN or serve files
directly from your web application.
The `rails_serv_static_assets` gem enables this behavior by setting the
`config.serve_static_assets` option to `true`, so you do not need to
configure it manually.

## <a id='buildpack'></a>Additional Ruby Buildpack Information ##

For information about using and extending the Ruby buildpack in Cloud Foundry,
see the [ruby-buildpack GitHub repo](https://github.com/cloudfoundry/ruby-buildpack).

You can find current information about this buildpack on the Ruby buildpack
[release page](https://github.com/cloudfoundry/ruby-buildpack/releases) in
GitHub.

The buildpack uses a default Ruby version of `2.2.2`.
To override this value for your app, add a Ruby declaration in the Gemfile.
This also applies to using a JRuby interpreter.

## <a id='env-var'></a>Environment Variables ##

You can access environments variable programmatically. For example, you can
obtain `VCAP_SERVICES` as follows:

```
ENV['VCAP_SERVICES']
```

Environment variables available to you include both those [defined by the system]
(../../devguide/deploy-apps/environment-variable.html#app-system-env)
and those defined by the Ruby buildpack, as described below.

### <a id='BUNDLE-BIN-PATH'></a>BUNDLE\_BIN\_PATH ###

Location where Bundler installs binaries.

`BUNDLE_BIN_PATH:/home/vcap/app/vendor/bundle/ruby/1.9.1/gems/bundler-1.3.2/bin/bundle`

### <a id='BUNDLE-GEMFILE'></a>BUNDLE_GEMFILE ###

Path to application's Gemfile.

`BUNDLE_GEMFILE:/home/vcap/app/Gemfile`

### <a id='BUNDLE-WITHOUT'></a>BUNDLE_WITHOUT ###

The `BUNDLE_WITHOUT` environment variable causes Cloud Foundry to skip
installation of gems in excluded groups.
`BUNDLE_WITHOUT` is particularly useful for Rails applications, where there are
typically “assets” and “development” gem groups containing gems that are not
needed when the app runs in production

For information about using this variable, see http://blog.cloudfoundry.com/2012/10/02/polishing-cloud-foundrys-ruby-gem-support.

`BUNDLE_WITHOUT=assets`

### <a id='DATABASE-URL'></a>DATABASE_URL ###

The Ruby buildpack looks at the database\_uri for bound services to see if they
match known database types.
If there are known relational database services bound to the application, the
buildpack sets up the DATABASE_URL environment variable with the first one in
the list.

If your application depends on DATABASE\_URL being set to the connection string
for your service, and Cloud Foundry does not set it, you can set this variable
manually.

`$ cf set-env my_app_name DATABASE_URL mysql://b5d435f40dd2b2:ebfc00ac@us-cdbr-east-03.cleardb.com:3306/ad_c6f4446532610ab`

### <a id='GEM-HOME'></a>GEM_HOME ###

Location where gems are installed.

`GEM_HOME:/home/vcap/app/vendor/bundle/ruby/1.9.1`

### <a id='GEM-PATH'></a>GEM_PATH ###

Location where gems can be found.

`GEM_PATH=/home/vcap/app/vendor/bundle/ruby/1.9.1:`

### <a id='RACK-ENV'></a>RACK_ENV ###
This variable specifies the Rack deployment environment: development,
deployment, or none.
This governs what middleware is loaded to run the application.

`RACK_ENV=production`

### <a id='RAILS-ENV'></a>RAILS_ENV ###
This variable specifies the Rails deployment environment: development, test, or
production.
This controls which of the environment-specific configuration files will govern
how the application will be executed.

`RAILS_ENV=production`

### <a id='RUBYOPT'></a>RUBYOPT ###
This Ruby environment variable defines command-line options passed to Ruby
interpreter.

`RUBYOPT: -I/home/vcap/app/vendor/bundle/ruby/1.9.1/gems/bundler-1.3.2/lib -rbundler/setup`
