---
title: Getting Started Deploying Ruby on Rails Apps
owner: Buildpacks
---

<strong></strong>

<%=vars.GSG_intro_sentence%>

This guide walks you through deploying a Ruby on Rails (RoR) app to <%=vars.product_full%>. To deploy a sample RoR app, refer to the [Deploy a Sample Ruby on Rails App](./sample-ror.html) topic.

<p class="note"><strong>Note</strong>: Ensure that your RoR app runs locally before continuing with this procedure.</p>

## <a id='prerequisites'></a>Prerequisites ##

* A Rails 4.x app that runs locally
* [Bundler](http://bundler.io) configured on your workstation
* Intermediate to advanced RoR knowledge
* The [cf Command Line Interface (CLI)](../../cf-cli/install-go-cli.html)

## <a id="service-instance"></a> Step 1: Create and Bind a Service Instance for a RoR Application ##

This section describes using the CLI to configure a PostgreSQL managed service instance for an app. For more information about creating and using service instances, refer to the [Services Overview](./../../devguide/services/index.html) topic.

### Create a Service Instance ###

Run `cf marketplace` to view managed and user-provided services and plans available to you.

Run `cf create-service SERVICE PLAN SERVICE_INSTANCE` to create a service instance for your app. Choose a SERVICE and PLAN from the list, and provide a unique name for the SERVICE_INSTANCE.

### Bind a Service Instance ###

When you bind an app to a service instance, <%=vars.product_short%> writes information about the service instance to the VCAP_SERVICES app environment variable. The app can use this information to integrate with the service instance.

Most services support bindable service instances. Refer to your service provider's documentation to confirm whether they support this functionality.

To bind a service to an application, run `cf bind-service APPLICATION SERVICE_INSTANCE`.


## <a id="deployment-options"></a> Step 2: Configure Deployment Options  ##

### Configure the Deployment Manifest ###

You can specify app deployment options in a manifest that the `cf push` command uses. For more information about application manifests and supported attributes, refer to the [Deploying with Application Manifests](./../../devguide/deploy-apps/manifest.html) topic.

### Configure a Production Server ###

<%=vars.product_short%> uses the default standard Ruby web server library, WEBrick, for Ruby and RoR apps. However, <%=vars.product_short%> can support a more robust production web server, such as Phusion Passenger, Puma, Thin, or Unicorn. If your app requires a more robust web server, refer to the [Configure a Production Server](./ruby-prod-server.html) topic for help configuring a server other than WEBrick.

## <a id="login"></a> Step 3: Log in and Target the API Endpoint ##

Run `cf login -a API_ENDPOINT`, enter your login credentials, and select a space and org. The API endpoint is <%=vars.api_endpoint%>.

  <%=vars.gen_GSG%>

## <a id="deploy"></a> Step 4: Deploy Your App ##

From the root directory of your application, run `cf push APP-NAME --random-route` to deploy your application.

`cf push APP-NAME` creates a URL route to your application in the form HOST.DOMAIN, where HOST is your APP-NAME and DOMAIN is specified by your administrator. Your DOMAIN is`<%=vars.app_domain%>`. For example: `cf push my-app` creates the URL `my-app.<%=vars.app_domain%>`.

The URL for your app must be unique from other apps that <%=vars.product_short%> hosts or the push will fail. Use the following options to help create a unique URL:

* `-n` to assign a different HOST name for the app.
* `--random-route` to create a URL that includes the app name and random words.
* `cf help push` to view other options for this command.

To view log activity while the app deploys, launch a new terminal window and run `cf logs APP-NAME`.

In the terminal output of the `cf push` command, the `urls` field of the `App started` block contains the app URL. This is the `HOST_NAME` you specified with the `-n` flag plus the domain `<%=vars.app_domain%>`. Once your app deploys, use this URL to access the app online.


##<a id="next"></a>Next Steps ##

You’ve deployed an app to <%=vars.product_short%>! Consult the sections below for information about what to do next.

### Test a Deployed App ###

Use the cf <%=vars.dev_console_2%> to review information and administer your app and your <%=vars.product_short%> account. For example, you could edit the `manifest.yml` to increase the number of app instances from 1 to 3, and redeploy the app with a new app name and host name.

### Manage Your Application with the cf CLI ###

Run `cf help` to view a complete list of commands, grouped by task categories, and run `cf help COMMAND` for detailed information about a specific command. For more information about using the cf CLI, refer to the cf Command Line Interface (CLI) topics, especially the <%=vars.cli_v6%> topic.


### Troubleshooting ###
If your application fails to start, verify that the application starts in your local environment. Refer to the [Troubleshooting Application Deployment and Health](./../../devguide/deploy-apps/troubleshoot-app-health.html) topic to learn more about troubleshooting.








