---
title: PHP Buildpack Configuration
owner: Buildpacks
---

<strong></strong>

## <a id="defaults"></a> Defaults

The buildpack stores all of its default configuration settings in the [defaults] directory.

## <a id="options"></a> options.json

The `options.json` file is the configuration file for the buildpack itself. It instructs the buildpack what to download, where to download it from, and how to install it. It allows you to configure package names and versions (i.e. PHP, HTTPD, or Nginx versions), the web server to use (HTTPD, Nginx, or None), and the PHP extensions that are enabled.

The buildpack overrides the default `options.json` file with any configuration it finds in the `.bp-config/options.json` file of your application.

Below is an explanation of the common options you might need to change.

| Variable                     | Explanation                                                                                                                                                                                                                                                                                                                                          |
| -------------------          | -----------------------------------------------------                                                                                                                                                                                                                                                                                                |
| WEB\_SERVER                  | Sets the web server to use. Must be one of `httpd`, `nginx`, or `none`. This value defaults to `httpd`.                                                                                                                                                                                                                                           |
| HTTPD\_VERSION                | Sets the version of Apache HTTPD to use. Currently the build pack supports the latest stable version.  This value will default to the latest release that is supported by the build pack. |
| ADMIN\_EMAIL                 | The value used in HTTPD's configuration for [ServerAdmin]                                                                                                                                                                                                                                                                                            |
| NGINX\_VERSION               | Sets the version of Nginx to use. By default, the buildpack uses the latest stable version.                                                                                                                                                                                                                      |
| PHP\_VERSION                 | Sets the version of PHP to use.                                                                                                                                                                             |
| PHP\_EXTENSIONS              | A list of the [extensions](#php-extensions) to enable.  `bz2`, `zlib`, `curl`, and `mcrypt` are enabled by default.                                                                                                                                                                                                                                 |
| PHP\_MODULES                 | A list of the [modules](#php-modules) to enable.  No modules are explicitly enabled by default, however the buildpack automatically chooses `fpm` or `cli`. You can explicitly enable any or all of: `fpm`, `cli`, `cgi`, and `pear`.                                                                                           |
| ZEND\_EXTENSIONS             | A list of the Zend extensions to enable. Nothing is enabled by default.                                                                                                                                                                                                                                                                           |
| APP\_START\_CMD              | When the `WEB_SERVER` option is set to 'none,' this command is used to start your app. If `WEB_SERVER` and `APP_START_CMD` are not set, then the buildpack searches for `app.php`, `main.php`, `run.php`, or `start.php` (in that order). This option accepts arguments. |
| WEBDIR                       | The root directory of the files served by the web server specified in `WEB_SERVER`. Defaults to `htdocs`. Other common settings are `public`, `static`, or `html`. Path is relative to the root of your application.           |
| LIBDIR                       | This path is added to PHP's `include_path`. Defaults to `lib`. Path is relative to the root of your application.           |
| HTTP\_PROXY                  | The buildpack downloads uncached dependencies using HTTP. If you are using a proxy for HTTP access, set its URL here.
| HTTPS\_PROXY                  | The buildpack downloads uncached dependencies using HTTPS. If you are using a proxy for HTTPS access, set its URL here.
| ADDITIONAL\_PREPROCESS\_CMDS | A list of additional commands that will run prior to the application starting. For example, you might use this command to run migration scripts or static caching tools before the application launches.

For details about supported versions, please read the [release notes](https://github.com/cloudfoundry/php-buildpack/releases) for your buildpack version.

### <a id="engine-configurations"></a> HTTPD, Nginx and PHP configuration

The buildpack automatically configures HTTPD, Nginx and PHP for your application.  This section explains how to modify the configuration.

The `.bp-config` directory in your application can contain configuration overrides for these components. Name the directories `httpd`, `nginx`, and `php`.

For example:
```
.bp-config
  httpd
  nginx
  php
```

Each directory can contain configuration files that the component understands.

For example, to change HTTPD logging configuration:

```
$ ls -l .bp-config/httpd/extra/
total 8
-rw-r--r--  1 daniel  staff  396 Jan  3 08:31 httpd-logging.conf
```

In this example, the `httpd-logging.conf` file overrides the one provided by the buildpack. We recommend that you copy the default from the buildpack and modify it.

The default configuration files are found in the [PHP Buildpack `/defaults/config` directory](https://github.com/cloudfoundry/php-buildpack/tree/master/defaults/config)

Take care when modifying configurations, as it might cause your application to fail, or cause Cloud Foundry to fail to stage your application.

You can add your own configuration files. The components will not know about these, so you must ensure that they are included. For example, you can add an include directive to the [httpd configuration](https://github.com/cloudfoundry/php-buildpack/blob/master/defaults/config/httpd/2.4.x/httpd.conf) to include your file:

```
ServerRoot "${HOME}/httpd"
Listen ${PORT}
ServerAdmin "${HTTPD_SERVER_ADMIN}"
ServerName "0.0.0.0"
DocumentRoot "${HOME}/#{WEBDIR}"
Include conf/extra/httpd-modules.conf
Include conf/extra/httpd-directories.conf
Include conf/extra/httpd-mime.conf
Include conf/extra/httpd-logging.conf
Include conf/extra/httpd-mpm.conf
Include conf/extra/httpd-default.conf
Include conf/extra/httpd-remoteip.conf
Include conf/extra/httpd-php.conf
Include conf/extra/httpd-my-special-config.conf # This line includes your additional file.
```

### <a id="php-extensions"></a> PHP Extensions

PHP extensions are easily enabled by setting the `PHP_EXTENSIONS` or `ZEND_EXTENSIONS` option in `.bp-config/options.json`.  Use these options to install bundled PHP extensions.

### <a id="php-modules"></a> PHP Modules

The following modules can be included by adding it to the `PHP_MODULES` list:
  - `cli`, installs `php` and `phar`
  - `fpm`, installs `PHP-FPM`
  - `cgi`, installs `php-cgi`
  - `pear`, installs Pear

  By default, the buildpack installs the `cli` module when you push a standalone application, and it installs the `fpm` module when you run a web application. You must specify `cgi` and `pear` if you want them installed.

## <a id="buildpack-extensions"></a>Buildpack Extensions

The buildpack comes with extensions for its default behavior. These are the [HTTPD], [Nginx], [PHP], and [NewRelic] extensions.

The buildpack is designed with an extension mechanism, allowing application developers to add behavior to the buildpack without modifying the buildpack code.

When an application is pushed, the buildpack runs any extensions found in the `.extensions` directory of your application.

The [Development Documentation] explains how to write extensions.

[defaults]:https://github.com/cloudfoundry/php-buildpack/tree/master/defaults
[ServerAdmin]:http://httpd.apache.org/docs/2.4/mod/core.html#serveradmin
[extra/httpd-logging.conf]:https://github.com/cloudfoundry/php-buildpack/blob/master/defaults/config/httpd/2.4.x/extra/httpd-logging.conf
[Development Documentation]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/development.md
[HTTPD]:https://github.com/cloudfoundry/php-buildpack/tree/master/lib/httpd
[Nginx]:https://github.com/cloudfoundry/php-buildpack/tree/master/lib/nginx
[PHP]:https://github.com/cloudfoundry/php-buildpack/tree/master/lib/php
[NewRelic]:https://github.com/cloudfoundry/php-buildpack/tree/master/extensions/newrelic
