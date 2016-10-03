---
title: Configure a Production Server for Ruby Apps
owner: Buildpacks
---

<strong></strong>

<%=vars.product_short%> uses the default standard Ruby web server library WEBrick for Ruby and RoR apps. However, <%=vars.product_short%> can support a more robust production web server, such as Phusion Passenger, Puma, Thin, or Unicorn.

To instruct <%=vars.product_short%> to use a web server other than WEBrick, you configure a text file called a _Procfile_. A Procfile enables you to declare required runtime processes, called _process_ _types_, for your web app. Process managers in a server use the process types to run and manage the workload.

When you deploy, <%=vars.product_short%> determines if a Procfile exists and uses the Procfile to configure your app with the specified production web server settings.

In a Procfile, you declare one process type per line and use the following syntax, as shown in the example below:

* `PROCESS_TYPE` is the command name in the format of an alphanumeric string. Specifically for RoR web apps, you can declare `web` and `worker` process types.
* `COMMAND` is the command line to launch the process.

Example process type syntax:
    <pre class="code">
    PROCESS_TYPE: COMMAND
</pre>

To set a different production web server for your app:

1. Add the gem for the web server to your Gemfile.

1. In the `config` directory of your app, create a new configuration file or modify an existing file.

    Refer to your web server documentation for how to configure this file.

    Example Puma config file:
    <pre class="code">
    &#35; config/puma.rb
    threads 8,32
    workers 3

    on\_worker\_boot do
      # things workers do
    end
</pre>

1. In the root directory of your app, create a Procfile and add a command line for a `web` process type that points to your web server.

    The example below shows a command that starts a Puma web server and specifies the app runtime environment, TCP port, and paths to the server state information and configuration files.
    Refer to your web server documentation for how to configure the specific command for a process type.

    Example Procfile:
    <pre class="code">
    web: bundle exec puma -e $RAILS_ENV -p 1234 -S ~/puma -C config/puma.rb
    </pre>
