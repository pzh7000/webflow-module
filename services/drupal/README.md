# Composer template for Drupal projects

[![Build Status](https://travis-ci.org/drupal-composer/drupal-project.svg?branch=8.x)](https://travis-ci.org/drupal-composer/drupal-project)

This project template provides a starter kit for managing your site
dependencies with [Composer](https://getcomposer.org/).

If you want to know how to use it as replacement for
[Drush Make](https://github.com/drush-ops/drush/blob/8.x/docs/make.md) visit
the [Documentation on drupal.org](https://www.drupal.org/node/2471553).

## Usage

First you need to [install composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx).

> Note: The instructions below refer to the [global composer installation](https://getcomposer.org/doc/00-intro.md#globally).
> You might need to replace `composer` with `php composer.phar` (or similar)
> for your setup.

After that you can create the project:

```
composer create-project forumone/drupal-project:9.x-dev some-dir --no-interaction
```

With `composer require ...` you can download new dependencies to your
installation.

```
cd some-dir
composer require drupal/devel:~1.0
```

The `composer create-project` command passes ownership of all files to the
project that is created. You should create a new git repository, and commit
all files not excluded by the .gitignore file.

### Configuring Capistrano

Configuring Capistrano deployments requires editing of two files:

- `capistrano/deploy.rb`
- `capistrano/deploy/dev.rb`

#### `capistrano/deploy.rb`

Configuring the general deployment settings happens in `capistrano/deploy.rb` and requires replacing the
following placeholder tokens in the settings:

##### `APP_NAME`

This is simply a name for the application that will be used as a directory name. Replace it with a relevant
string to be used to identify your application.

##### `HTTPS_REPO_URL`

This should be the HTTPS clone URL for your repo to be deployed. You may access this from the GitHub UI.

#### `capistrano/deploy/dev.rb`

The files located at `capistrano/deploy/*.rb` define deployment targets for the application to be released to.
For each environment the application is being deployed to there should be one matching file with the
environment-specific configuration defined. To create a new deployment environment, the
`capistrano/deploy/dev.rb` file may be duplicated and renamed to match the name of the new environment. Then
the same configuration process described below should be followed by customizing each of the following tokens
in the file settings:

##### `stage`

The stage name should match that of the containing file. For example, the `dev.rb` file should set this to `:dev`.

##### `SITE_URL`

This is the URL used to access the site being deployed.

##### `ENVIRONMENT_NAME`

This is the name of the environment's vhost directory where the application will be deployed to. Usually this
is a combination of a short application name followed by the environment name, e.g., `drupal.dev`. By replacing
this token in the full path, the setting would look like this:

```ruby
# The path to the project on the server
set :deploy_to, '/var/www/vhosts/drupal.dev'
```

##### `BRANCH`

This is the specific git branch to be deployed to this environment from the repository. Typically these follow the pattern in the following table.

| Environment | Branch        |
| ----------- | ------------- |
| dev         | `integration` |
| stage       | `main`        |
| prod        | `live`        |

##### `SERVER_LOGIN`

This defines the servers to be deployed to and the logins to be used for access. In most use cases, each
instance of this token will use the same login. An example login would look like:

```ruby
# Simple Role Syntax
# ==================
# Supports bulk-adding hosts to roles, the primary
# server in each group is considered to be the first
# unless any hosts have the primary property set.
role :app, %w{drupal@drupal.byf1.dev}, :primary => true
role :web, %w{drupal@drupal.byf1.dev}
role :db,  %w{drupal@drupal.byf1.dev}
```

## What does the template do?

When installing the given `composer.json` some tasks are taken care of:

- Drupal will be installed in the `web`-directory.
- Autoloader is implemented to use the generated composer autoloader in `vendor/autoload.php`,
  instead of the one provided by Drupal (`web/vendor/autoload.php`).
- Modules (packages of type `drupal-module`) will be placed in `web/modules/contrib/`
- Theme (packages of type `drupal-theme`) will be placed in `web/themes/contrib/`
- Profiles (packages of type `drupal-profile`) will be placed in `web/profiles/contrib/`
- Creates default writable versions of `settings.php` and `services.yml`.
- Creates `web/sites/default/files`-directory.
- Latest version of drush is installed locally for use at `vendor/bin/drush`.
- Latest version of DrupalConsole is installed locally for use at `vendor/bin/drupal`.
- Creates environment variables based on your .env file. See [.env.example](.env.example).

## Updating Drupal Core

This project will attempt to keep all of your Drupal Core files up-to-date; the
project [drupal/core-composer-scaffold](https://github.com/drupal/core-composer-scaffold)
is used to ensure that your scaffold files are updated every time drupal/core is
updated. If you customize any of the "scaffolding" files (commonly .htaccess),
you may need to merge conflicts if any of your modified files are updated in a
new release of Drupal core.

Follow the steps below to update your core files.

1. Run `composer update drupal/core drupal/core-dev --with-dependencies` to update Drupal Core and its dependencies.
2. Run `git diff` to determine if any of the scaffolding files have changed.
   Review the files for any changes and restore any customizations to
   `.htaccess` or `robots.txt`.
3. Commit everything all together in a single commit, so `web` will remain in
   sync with the `core` when checking out branches or running `git bisect`.
4. In the event that there are non-trivial conflicts in step 2, you may wish
   to perform these steps on a branch, and use `git merge` to combine the
   updated core files with your customized files. This facilitates the use
   of a [three-way merge tool such as kdiff3](http://www.gitshah.com/2010/12/how-to-setup-kdiff-as-diff-tool-for-git.html). This setup is not necessary if your changes are simple;
   keeping all of your modifications at the beginning or end of the file is a
   good strategy to keep merges easy.

## Generate composer.json from existing project

With using [the "Composer Generate" drush extension](https://www.drupal.org/project/composer_generate)
you can now generate a basic `composer.json` file from an existing project. Note
that the generated `composer.json` might differ from this project's file.

## FAQ

### Should I commit the contrib modules I download?

Composer recommends **no**. They provide [argumentation against but also
workrounds if a project decides to do it anyway](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md).

### Should I commit the scaffolding files?

The [Drupal Composer Scaffold](https://github.com/drupal/core-composer-scaffold) plugin can download the scaffold files (like
index.php, update.php, …) to the web/ directory of your project. If you have not customized those files you could choose
to not check them into your version control system (e.g. git). If that is the case for your project it might be
convenient to automatically run the drupal-scaffold plugin after every install or update of your project. You can
achieve that by registering `@composer drupal:scaffold` as post-install and post-update command in your composer.json:

```json
"scripts": {
    "post-install-cmd": [
        "@composer drupal:scaffold",
        "..."
    ],
    "post-update-cmd": [
        "@composer drupal:scaffold",
        "..."
    ]
},
```

### How can I apply patches to downloaded modules?

If you need to apply patches (depending on the project being modified, a pull
request is often a better solution), you can do so with the
[composer-patches](https://github.com/cweagans/composer-patches) plugin.

To add a patch to drupal module foobar insert the patches section in the extra
section of composer.json:

```json
"extra": {
    "patches": {
        "drupal/foobar": {
            "Patch description": "URL or local path to patch"
        }
    }
}
```

### How do I switch from packagist.drupal-composer.org to packages.drupal.org?

Follow the instructions in the [documentation on drupal.org](https://www.drupal.org/docs/develop/using-composer/using-packagesdrupalorg).

### How do I specify a PHP version ?

This project supports PHP 7.0 as minimum version (see [Drupal 8 PHP requirements](https://www.drupal.org/docs/8/system-requirements/drupal-8-php-requirements)), however it's possible that a `composer update` will upgrade some package that will then require PHP 7+.

To prevent this you can add this code to specify the PHP version you want to use in the `config` section of `composer.json`:

```json
"config": {
    "sort-packages": true,
    "platform": {
        "php": "7.0.33"
    }
},
```
