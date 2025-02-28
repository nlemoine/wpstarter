---
title: WordPress Integration
nav_order: 3
---

# WordPress Integration
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

## Include WordPress in the project

Considering that WordPress has no official support for Composer, there's also no official way to integrate WordPress with Composer.

One way to do it is **treating WordPress as a dependency**, like any other dependency.

To date, WordPress does not officially provide a Composer compatible repository of WordPress core (basically having a `composer.json`).

The most used non-official package with Composer support is maintained by [Roots](https://roots.io/), that has almost multiple millions of downloads from [packagist.org](https://packagist.org/packages/roots/wordpress).

That said, **WP Starter does not declare any of those packages** as a dependency, allowing for the use of custom packages or even bypassing the  installation of WordPress entirely.

For example, an alternative way could be to use Composer [repositories](https://getcomposer.org/doc/05-repositories.md) settings to build a custom package using the official zip distribution:

```json
{
  "name": "my-company/my-project",
  "repositories": [
    {
      "type": "package",
      "package": {
        "name": "wordpress/wordpress",
        "version": "6.1.1",
        "dist": {
          "url": "https://wordpress.org/wordpress-6.1.1-no-content.zip",
          "type": "zip"
        }
      }
    }
  ],
  "require": {
    "wecodemore/wpstarter": "^3",
    "wordpress/wordpress": "6.1.1"
  }
}
```

Yet more ways to install WordPress could include using a [custom package](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md), or make use of the [wp-downloader](https://github.com/wecodemore/wp-downloader) Composer plugin.

Please note: if WordPress is **not** installed as Composer package (and that's the case when using [wp-downloader](https://github.com/wecodemore/wp-downloader)) then it is necessary to set WP Starter setting `{ "require-wp": false }` to prevent WP Starter complaining about not WordPress found.


### Dealing with default content

When WordPress is installed using a core package like the [one](https://packagist.org/packages/johnpbloch/wordpress) from [John P. Bloch](https://johnpbloch.com/), the package comes with default themes and plugins ( "Hello Dolly").

In WP Starter installations the "wp-content" folder is a separate folder, and so themes and plugins hat comes with a WordPress package are not recognized.

WP Starter provides an option to register default themes shipped with core packages, so that they can be recognized:

```json
{
    "extra": {
        "wpstarter": {
            "register-theme-folder": true
        }
    }
}
```

There's also the `move-content` option that when set to `true` tells WP Starter to moves the entire `wp-content` folder shipped with a WordPress package into the project's content folder.

If there's no interest in default plugin/themes, the suggested approach is to use the [Root's package](https://roots.io/), [wp-downloader](https://github.com/wecodemore/wp-downloader), or a custom package that points to a "*no-content*" WordPress distribution.



## WP Starter `wp-config` interactions

When WP Starter finishes its job it leaves the project folder ready to be used, but WP Starter code does **not** interact with WordPress after its job is done.

However, the `wp-config.php` that WP Starter generates it is a different from the standard one, and introduces a few WP Starter-specific functionalities.

It is important to say that everything in this section will refer to the `wp-config.php` file generated by WP Starter using _default template_, but as better explained in the [*"WP Starter Steps"* chapter](05-WP-Starter-Steps.md), WP Starter templates can be customized, and if that happens for the `wp-config.php` template, there's no way to tell if the functionalities described in this chapter will be there or not.



### Environment

In the [*"Environment Variables"* chapter](02-Environment-Variables.md) we presented how WP Starter sets WordPress constants from env variables. WP Starter does it via code placed in the  `wp-config.php` generated by WP Starter.

Beside the environment variables named after WordPress constants, there are a few more env variables that have a special meaning for WP Starter WordPress installations.

#### WP_ENVIRONMENT_TYPE / WP_ENV

`WP_ENVIRONMENT_TYPE` is an environment variable supported by WordPress, and can also be used by WP Starter to determine the current application environment, for example "*production*", "*staging*", and so on.

Support for `WP_ENVIRONMENT_TYPE` environment variable was added in WordPress core with version 5.5, and WP Starter started using it since then.

Before that, WP Starter used the environment variable `WP_ENV`, and `WORDPRESS_ENV` was used in WP Starter v1.

For backward compatibility reasons, both `WP_ENV` and `WORDPRESS_ENV` are still supported, and in case those are used, WP Starter will _also_ set the `WP_ENVIRONMENT_TYPE` constant to ensure compatibility with WordPress 5.5+.

This means that projects that are already using `WP_ENV` or `WORDPRESS_ENV` can upgrade to latest WP Starter without any need to change environment variables and get support for latest WordPress.

##### WP Starter environment VS WordPress environment

WordPress does not allow arbitrary values for `WP_ENVIRONMENT_TYPE`, in fact it  requires the value of that variable to be one of:

- `"local"`
- `"development"`
- `"staging"`
- `"production"`

Setting `WP_ENVIRONMENT_TYPE` to anything else, will make [`wp_get_environment_type()`](https://developer.wordpress.org/reference/functions/wp_get_environment_type/) function return the default "*production*".

Unlike WordPress, WP Starter does not limit environment to specific values, and in the case a value not supported by WordPress is used, to maximize compatibility, WP Starter will try to "map" different values to one of those supported by WP.

For example, setting `WP_ENVIRONMENT_TYPE` env variable to `"develop"` WP Starter will define a `WP_ENVIRONMENT_TYPE` constant having `"development"` as value.

The original `"develop"` value will be available in the `WP_ENV` constant.

In the case WP Starter is not able to map an environment to a value supported by WordPress, the original value will be available in both `WP_ENVIRONMENT_TYPE` and `WP_ENV`, but the WordPress [`wp_get_environment_type`](https://developer.wordpress.org/reference/functions/wp_get_environment_type/)  function will return `"production"` because that is the default value used by WordPress.

Let's clarify with an example. If a project sets an env variable like this:

```bash
WP_ENV=preprod
```

WP Starter will declare:

```php
define('WP_ENVIRONMENT_TYPE', 'staging');
define('WP_ENV', 'preprod');
```

WP Starter was able to map `"preprod"` to the WordPress-supported `"staging"` value, and thanks to that, `wp_get_environment_type()` will correctly return `"staging"`.

The "mapping" happens by looking for alias according to the following table:

| `WP_ENVIRONMENT_TYPE `/ `WP_ENV` | WP-supported value |
|----------------------------------|--------------------|
| dev                              | development        |
| develop                          | development        |
| stage                            | staging            |
| pre                              | staging            |
| preprod                          | staging            |
| pre-prod                         | staging            |
| pre-production                   | staging            |
| preproduction                    | staging            |
| test                             | staging            |
| tests                            | staging            |
| testing                          | staging            |
| uat                              | staging            |
| qa                               | staging            |
| acceptance                       | staging            |
| accept                           | staging            |
| prod                             | production         |
| live                             | production         |
| public                           | production         |

Moreover, if the  `WP_ENVIRONMENT_TYPE` (`WP_ENV` / `WORDPRESS_ENV`) environment variable value _contains_ one of the supported values, it will be mapped to it, for example `"production-1"` will be mapped to `"production"`  or `"uat-us-1"` will be mapped to `"staging"`.

**Please note**: when in this docs we refer to _WP Starter environment_ we refer to what's defined in "original" value defined in `WP_ENVIRONMENT_TYPE`/`WP_ENV`/ `WORDPRESS_ENV` and not to the "mapped" value. That is relevant for "environment-specific" files, that will be introduced below.

Another example. If a project sets an environment variable like this:

```bash
WP_ENV=something_very_custom
```

WP Starter will declare:

```php
define('WP_ENVIRONMENT_TYPE', 'production');
define('WP_ENV', 'something_very_custom');
```

WP Starter was not able to map `"something_very_custom"` to any of the four environment types supported by WordPress, so it stored "*production*" in `WP_ENVIRONMENT_TYPE` constant, because that's the WordPress default.

It must be noted that is possible to use a custom WP Starter-specific environment and a WordPress-compatible environment by setting _both_ `WP_ENV` and `WP_ENVIRONMENT_TYPE`. 

For example, having an environment like this:

```bash
WP_ENV=something_very_custom
WP_ENVIRONMENT_TYPE=development
```

WP Starter will declare:

```php
define('WP_ENVIRONMENT_TYPE', 'development');
define('WP_ENV', 'something_very_custom');
```



##### Environment-specific files

After environment variables are loaded (via either _actual_ environment or via env file) WP Starter will look, in the same directory where it looks for "main" env file (by default project root), for two environment-specific files whose name depends on the value of `WP_ENV` constant.

They are:

- an **env file** named  **`{$envFile}.{$environment}`**, where `$envFile` is the name of the "main" env file (by default `.env`) and `$environment` is the value of  `WP_ENV`.
- a **PHP file** named like **`{$environment}.php`**, where `$environment` is the value of `WP_ENV` constant.

###### Environment-specific env file

If the environment-specific env file is found, it will be loaded, and all the env variables defined there will be merged with anything already loaded (via either actual environment or via main env file).

Note that environment-specific env file can overwrite variables in main env file, but variables defined in actual environment will always "win" over variables defined in env files.

###### Environment-specific PHP file

If the environment-specific PHP file is found it is just included. This allows for advanced per-environment settings.

To make such an configuration that involves WordPress, the `{$environment}.php` file will likely need to use WordPress hooks, and that is possible because WP Starter loads the `wp-includes/plugin.php` early, before the rest of WP is loaded. More on this below.



##### Default environments

If the WP Starter environment is one of the following values:

- `"local"`
- `"development"`
- `"staging"`
- `"production"`

WP Starter will setup WordPress debug-related PHP constants accordingly.

The code that does that looks like this:

```php
switch ($environment) {
    case 'local':
        defined('WP_LOCAL_DEV') or define('WP_LOCAL_DEV', true);
        // no break
    case 'development':
        defined('WP_DEBUG') or define('WP_DEBUG', true);
        defined('WP_DEBUG_DISPLAY') or define('WP_DEBUG_DISPLAY', true);
        defined('WP_DEBUG_LOG') or define('WP_DEBUG_LOG', false);
        defined('SAVEQUERIES') or define('SAVEQUERIES', true);
        defined('SCRIPT_DEBUG') or define('SCRIPT_DEBUG', true);
        defined('WP_DISABLE_FATAL_ERROR_HANDLER') or define('WP_DISABLE_FATAL_ERROR_HANDLER', true);
        break;
    case 'staging':
        defined('WP_DEBUG') or define('WP_DEBUG', true);
        defined('WP_DEBUG_DISPLAY') or define('WP_DEBUG_DISPLAY', false);
        defined('WP_DEBUG_LOG') or define('WP_DEBUG_LOG', true);
        defined('SAVEQUERIES') or define('SAVEQUERIES', false);
        defined('SCRIPT_DEBUG') or define('SCRIPT_DEBUG', true);
        break;
    case 'production':
    default:
        defined('WP_DEBUG') or define('WP_DEBUG', false);
        defined('WP_DEBUG_DISPLAY') or define('WP_DEBUG_DISPLAY', false);
        defined('WP_DEBUG_LOG') or define('WP_DEBUG_LOG', false);
        defined('SAVEQUERIES') or define('SAVEQUERIES', false);
        defined('SCRIPT_DEBUG') or define('SCRIPT_DEBUG', false);
        break;
}
```



#### Cached Environment

Parsing the environment and define constants for it can be quite expensive process. Thanks to code placed in `wp-config.php` that runs on "shutdown" the environment is cached in a file named `.env.cached.php` that contains PHP code that declares env variables and defines constants.

Details about this process and a way to prevent this are described in the [*"Environment-Variables"* chapter](02-Environment-Variables.md).



#### WP_HOME

This is not a WP Starter variable, but a standard WordPress configuration constant.

However, unlike WordPress, WP Starter will always make sure it is set.

If no environment variable with that name is found, WP Starter will calculate the home URL by looking at server variables, defaulting to `"localhost"` if server variables are not found.

This might be fine in many cases, but **setting `WP_HOME` is highly recommended** to avoid issues and avoid calculation of the home URL at every request.



#### WP_ADMIN_COLOR

`WP_ADMIN_COLOR` environment variable will make WP Starter add a filter in the generated `wp-config.php` that will force WordPress dashboard to a specific admin color scheme, overriding the user setting.

For example:

```shell
WP_ADMIN_COLOR=ectoplasm
```

Will force the current environment to use the "ectoplasm" admin color scheme.

Supported schemes are the ones shipped core ("light", "blue", "coffee", "ectoplasm", "midnight", "ocean", "sunrise") plus any additional color scheme that plugins might have added to WordPress.

Changing admin color per environment helps to visually recognize which is the current environment and avoid, for example, doing in production operations meant for staging.



### Early loading of `plugin.php`

The file `wp-includes/plugin.php` contains WordPress functions for the [plugin API](https://developer.wordpress.org/plugins/hooks/).

In recent WordPress versions this file has been made "independent" from the rest of WordPress, which means that it can be loaded very early, allowing to add hooks before the rest of WordPress is loaded.

In the `wp-config.php` that WP Starter generates, the file is loaded very early making possible to add hooks, for example, in the environment-specific  `{$environment}.php` files, or in a dedicated "early hooks file".



### Early hook file

In WP Starter configuration it is possible to set an "early hooks file" loaded very early (right after `{$environment}.php`) that can be used to add callbacks to hooks triggered very early, like for example [`"enable_loading_advanced_cache_dropin"`](https://developer.wordpress.org/reference/hooks/enable_loading_advanced_cache_dropin/) or to set just-in-time WordPress configuration PHP constants.



### HTTPs behind load-balancers

When websites are behind load-balancers the server variable `$_SERVER['HTTPS']` is sometimes not set, even if the website is implementing HTTPS, and because WordPress function [`is_ssl()`](https://developer.wordpress.org/reference/functions/is_ssl/) relies on that server variable it returns `false` in that case, with many side effects.

As quite standard practice, in the above situations, load balancers set the server variable `$_SERVER['HTTP_X_FORWARDED_PROTO']` to `'https'` . So by looking at that value the result of `is_ssl()` could be forced to `true` by forcing  `$_SERVER['HTTPS']` to `'on'`.

WP Starter can do that. To enable this feature it is necessary to set the environment variable **`WP_FORCE_SSL_FORWARDED_PROTO`** to true.

There might be security implications in this, please see the "Make WordPress" ticket [#31288](https://core.trac.wordpress.org/ticket/31288) for details.

The gist of it is that `HTTP_X_FORWARDED_PROTO` is a server variable filled from an a HTTP header (like any `$_SERVER` variable whose name starts with `HTTP_`) which means that it could be set by the client or by a [MITM](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) proxy. On the other hand, environment variable can only be set on the server, so checking the env variable makes the presence of HTTP header trustable.



### Advanced customization of WP Starter `wp-config.php`

All the features described above are applied via using a `wp-config.php` template that comes with WP Starter.

It is possible to have a completely custom template, but sometimes what it is desired is just a small customization, like adding a line of code. Using a custom template for that is not a good approach, because to have the same functionalities it would be necessary to copy the original template code, hence loosing all the improvements and fixes that newer versions of WP Starter could bring to the template.

This is why the WP Starter default `wp-config.php` template supports the concept of "sections".

A section is a portion of the configuration file delimited with "labels" with braces, for example:

```php
CLEAN_UP : {
    unset($envName, $envLoader, $cacheEnv);
} #@@/CLEAN_UP
```

The above snippet represents the code for a section named _"CLEAN_UP"_.

WP Starter ships an object called `WpConfigSectionEditor` that can be used to edit any of the section by appending, pre-pending, or replacing the section content. This is usually done in a custom step. 

Detailed documentation about custom steps development is documented in the [*"Custom Steps Development"* chapter](08-Custom-Steps-Development.md), but what's important here is that using a code like this:

```php
/** @var WeCodeMore\WpStarter\Util\WpConfigSectionEditor $editor */
$editor->append('CLEAN_UP', 'define("SOME_CONSTANT", TRUE);');
```

the section snippet above would become:

```php
CLEAN_UP : {
    unset($envName, $envLoader, $cacheEnv);
    define("SOME_CONSTANT", TRUE);
} #@@/CLEAN_UP
```

Besides `WpConfigSectionEditor::append()`, the object also has the `prepend`, `replace`, and `delete` methods, to, respectively, pre-pend, replace and delete the code in the given section.

Please note that the PHP code to append/prepend/replace is not checked in any way, so be sure that it is valid PHP code.

An instance of `WpConfigSectionEditor` can be obtained via the `Locator` class, documentation for it is in the [*"Custom Steps Development"* chapter](08-Custom-Steps-Development.md).



## Admin URL

When using WP Starter, the suggested way to install WordPress is to use a separate folder for core files.

The  [*"A Sample `composer.json`"* chapter](06-A-Commented-Sample-Composer-Json.md) shows an example on how to obtain it. 

When that is done, the `admin_url` becomes something like `example.com/wp/wp-admin/` (assuming WordPress is installed in the "/wp" folder).

It might be desirable to "rewrite" this URL to the usual `/wp-admin/` removing the subfolder. This is something that can be obtained via web-server configuration.

An example `.htaccess` file that contains rewrite rules for WordPress projects (compatible with multisite) can be found here: https://github.com/inpsyde/wpstarter-boilerplate/blob/master/templates/.htaccess.example.

Please note that using such configuration usually requires to set `WP_SITEURL` environment explicitly.



## Site Health / Info

In recent versions, WordPress has introduced a ["Site Health Screen"](https://wordpress.org/documentation/article/site-health-screen/) that offers a diagnosis of your site’s health. 

That is an admin screen with two tabs: "Status" and "Info".

The "status" tab display critical information about WordPress configuration. The "info" tab contains details about the configuration of the site.

WP Starter extends the "Info" tab providing a way to look into WP Starter configuration from the WP dashboard.

![site health info tab](img\site-info.png)



------

**Next:** [WP Starter Configuration](04-WP-Starter-Configuration.md)

---

- [Introduction](01-Introduction.md)
- [Environment Variables](02-Environment-Variables.md)
- ***WordPress Integration***
- [WP Starter Configuration](04-WP-Starter-Configuration.md)
- [WP Starter Steps](05-WP-Starter-Steps.md)
- [A Commented Sample `composer.json`](06-A-Commented-Sample-Composer-Json.md)
- [WP CLI Commands](07-WP-CLI-Commands.md)
- [Custom Steps Development](08-Custom-Steps-Development.md)
- [Settings Cheat Sheet](09-Settings-Cheat-Sheet.md)
- [Command-line Interface](10-Command-Line-Interface.md)

