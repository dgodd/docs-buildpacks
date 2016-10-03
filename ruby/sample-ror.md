---
title: Deploy a Sample Ruby on Rails Application
owner: Buildpacks
menu:
  main:
    Name: Deploy a Sample Ruby on Rails Application
    identifier: buildpacks/ruby/sample-ror
    parent: buildpacks/ruby
---



This topic guides the reader through deploying a sample Ruby on Rails app to <%= vars.product_full %> <%= vars.product_abbr %>.

##<a id='prerequisites'></a>Prerequisites

In order to deploy a sample Ruby on Rails app, you must have the following:

- A working <%=vars.product_name %> [deployment](http://docs.cloudfoundry.org/deploying/index.html)
- [Cloud Foundry CLI](../../cf-cli/install-go-cli.html)
- Cloud Foundry username and password with **Space Developer** [permissions](../../concepts/roles.html#space-roles). See your [Org Manager](../../concepts/roles.html#org-roles) if you require permissions.

##<a id='clone'></a>Step 1: Clone the App ##

Run the following terminal command to create a local copy of the rails\_sample\_app.
<pre class='terminal'>
	$ git clone https://github.com/cloudfoundry-samples/rails_sample_app
</pre>
The newly created directory contains a `manifest.yml` file, which assists CF with deploying the app. See [Deploying with Application Manifests](../../devguide/deploy-apps/manifest.html#minimal-manifest) for more information.

##<a id='login'></a>Step 2: Log in and Target the API Endpoint ##

1. Run the following terminal command to log in and target the API endpoint of your deployment.  <%= vars.api_endpoint_book %>

	<pre class='terminal'>$ cf login -a YOUR-API-ENDPOINT</pre>

1.	Use your credentials to log in, and to select a [Space and Org](../../concepts/roles.html).

<p class='note'><strong>Note</strong>: The API endpoint must be entered in the following format: `https://api.IP-ADDRESS`.


##<a id='service'></a>Step 3: Create a Service Instance ##

Run the following terminal command to create a PostgreSQL service instance for the sample app. Our service instance is `rails-postgres`. It uses the [`elephantsql`](http://www.elephantsql.com/) service and the `turtle` plan.
<pre class="terminal">
$   cf create-service elephantsql turtle rails-postgres
Creating service rails-postgres in org YOUR-ORG / space development as clouduser@example.com....
OK</pre>

The manifest for the rails\_sample\_app contains a `services` sub-block in the `applications` block. The cf Command Line Interface tool (cf CLI) binds the service to the app.

<pre class="terminal">
---
applications:
- name: rails-sample
  memory: 256M
  instances: 1
  path: .
  command: bundle exec rake db:migrate && bundle exec rails s -p $PORT
  services:
    - rails-postgres
</pre>

##<a id='deploy'></a>Step 4: Deploy the App ##

Make sure you are in the `rails_sample_app` directory. Run the following terminal command to deploy the app:
		<pre class='terminal'>$ cf push rails\_sample\_app</pre>

`cf push rails_sample_app` creates a URL route to your application in the form HOST.DOMAIN. In this example, HOST is rails\_sample\_app. Administrators specify the DOMAIN. For example, for the DOMAIN `<%=vars.app_domain%>`, running `cf push rails_sample_app` creates the URL `rails_sample_app.<%=vars.app_domain%>`.

The example below shows the terminal output when deploying the `rails_sample_app`. `cf push` uses the instructions in the manifest file to create the app, create and bind the route, and upload the app. It then binds the app to the `rails-postgres` service and follows the information in the manifest to start one instance of the app with 256M of RAM. After the app starts, the output displays the health and status of the app.


<pre class="terminal">

$ cf push rails_sample_app
Using manifest file ~/workspace/rails_sample_app/manifest.yml

Updating app rails_sample_app in org Cloud-Apps / space development as clouduser@example.com...
OK

Using route rails_sample_app.<%=vars.app_domain%>
Uploading rails_sample_app...
Uploading app files from: ~/workspace/rails_sample_app
Uploading 445.7K, 217 files
OK
Binding service rails-postgres to app rails_sample_app in org Cloud-Apps / space development as clouduser@example.com...
OK

Starting app rails_sample_app in org Cloud-Apps / space development as clouduser@example.com...
OK

...

0 of 1 instances running, 1 starting
1 of 1 instances running

App started

Showing health and status for app rails_sample_app in org Cloud-Apps / space development as clouduser@example.com...
OK

requested state: started
instances: 1/1
usage: 256M x 1 instances
urls: rails_sample_app.<%=vars.app_domain%>

     state     since                    cpu    memory          disk
#0   running   2014-08-25 03:32:10 PM   0.0%   68.4M of 256M   73.4M of 1G
</pre>

<p class='note'><strong>Note</strong>: If you want to view log activity while the app deploys, launch a new terminal window and run <code>cf logs rails\_sample\_app</code>.



##<a id='verify'></a> Step 5: Verifying the App ##

Verify that the app is running by browsing to the URL generated in the output of the previous step. In this example, navigating to `rails_sample_app.<%=vars.app_domain%>` verifies that the app is running.


You've now pushed an app to <%=vars.product_short%>! For more information on this topic, see the [Deploy an Application](../../devguide/deploy-apps/deploy-app.html) topic.
