# Composer template for Drupal projects

## Usage

First you need to [install composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx).

> Note: The instructions below refer to the [global composer installation](https://getcomposer.org/doc/00-intro.md#globally).
You might need to replace `composer` with `php composer.phar` (or similar) 
for your setup.

After that you can install all vendor packages (this includes Drupal Core and Contrib) and prepare the project file structure. 
After downloading the vendor packages some site specific questions will be asked (if necessary).

```
git clone https://github.com/Triquanta/drupal-project.git
mv drupal-project <project_name>
cd <project_name>
composer install
```

With `composer require ...` you can download new dependencies to your 
installation.

```
composer require drupal/diff:8.*
```

## What does the template do?

When installing the given `composer.json` some tasks are taken care of:

* Drupal will be installed in the `docroot`-directory.
* A site folder and files structure will be generated following Drupal's multi-site setup.
* Access to Drupal's `default` site is blocked.
* Autoloader is implemented to use the generated composer autoloader in `vendor/autoload.php`,
  instead of the one provided by Drupal (`docroot/vendor/autoload.php`).
* Modules (packages of type `drupal-module`) will be placed in `docroot/modules/contrib/`
* Theme (packages of type `drupal-theme`) will be placed in `docroot/themes/contrib/`
* Profiles (packages of type `drupal-profile`) will be placed in `docroot/profiles/contrib/`
* Creates a default writable version of `settings.php`.
* Creates `docroot/sites/{{ site_name }}/files`-directory.
* Creates environment specific `settings` and `services` files in the `docroot/sites/{{ site_name }}`-directory.
* Creates site specific database settings outside the `docroot`-directory in the `settings`-directory.
* Creates a site specific `sites.php` file in the `docrtoos/sites`-directory.
* Creates `config`-directory.
* Latest version of drush is installed locally for use at `vendor/bin/drush`.
* Creates a `drushrc.php` file with a default Drush `-l` argument to simplify Drush usage for a single site using a Drupal multi-site setup.
* Creates a site specific `aliases` file in the `drush`-directory.
* Removes `.txt` files in the `docroot/core`-directory.
* Latest version of DrupalConsole is installed locally for use at `vendor/bin/drupal`.

## Updating Drupal Core

This project will attempt to keep all of your Drupal Core files up-to-date; the 
project [drupal-composer/drupal-scaffold](https://github.com/drupal-composer/drupal-scaffold) 
is used to ensure that your scaffold files are updated every time drupal/core is 
updated. If you customize any of the "scaffolding" files (commonly .htaccess), 
you may need to merge conflicts if any of your modfied files are updated in a 
new release of Drupal core.

Follow the steps below to update your core files.

1. Run `composer update drupal/core`.
1. Run `git diff` to determine if any of the scaffolding files have changed. 
   Review the files for any changes and restore any customizations to 
  `.htaccess` or `robots.txt`.
1. Commit everything all together in a single commit, so `docroot` will remain in
   sync with the `core` when checking out branches or running `git bisect`.
1. In the event that there are non-trivial conflicts in step 2, you may wish 
   to perform these steps on a branch, and use `git merge` to combine the 
   updated core files with your customized files. This facilitates the use 
   of a [three-way merge tool such as kdiff3](http://www.gitshah.com/2010/12/how-to-setup-kdiff-as-diff-tool-for-git.html). This setup is not necessary if your changes are simple; 
   keeping all of your modifications at the beginning or end of the file is a 
   good strategy to keep merges easy.

## Listing the available build commands

You can get a list of all the available composer build commands with:

```
$ composer list
```

## Creating / adding a (multi) site

If you don't want to run `composer update/install` you can manually run the
script that prepares an environment specific site and file structure by
executing:

```
$ composer run-script post-install-cmd
```

Note, Drupals multi-site setup is used, but the script doesn't work 100% for
multiple sites. So review sites.php after adding another site.

## Set up tools for the development environment

If you chose `dev` as environment during `composer install/update`

This will perform the following tasks:

1. @todo Configure Behat.
1. @todo Configure PHP CodeSniffer.
1. Enable 'development mode'. This will:
  * Enable the services in `services.dev.yml`.
  * Show all error messages with backtrace information.
  * Disable CSS and JS aggregation.
  * Disable the render cache.
  * Allow test modules and themes to be installed.
  * Enable access to `rebuild.php`.

## Install the website.

A fresh standard Drupal site can be installed by executing:

```
$ composer drupal-install
```

Note: make sure you already have a working database, setup with credentials as
given during `composer update/install`.

Database settings will be read from file and account credentials can be chosen.

If you chose a `dev` environment the following tasks will also be performed:

1. @todo Enable development modules.
1. @todo Create a demo user for each user role.

## Running Behat tests

The Behat test suite is located in the `tests/` folder. The easiest way to run
them is by going into this folder and executing the following command:

```
$ cd tests/
$ ./behat
```

If you want to execute a single test, just provide the path to the test as an
argument. The tests are located in `tests/features/`:

```
$ cd tests/
$ ./behat features/authentication.feature
```

If you want to run the tests from a different folder, then provide the path to
`tests/behat.yml` with the `-c` option:

```
# Run the tests from the root folder of the project.
$ ./vendor/bin/behat -c tests/behat.yml
```


## Checking for coding standards violations

### Set up PHP CodeSniffer

PHP CodeSniffer is included to do coding standards checks of PHP and JS files.
In the default configuration it will scan all files in the following folders:
- `docroot/modules` (excluding `docroot/modules/contrib`)
- `docroot/profiles`
- `docroot/themes`

First you'll need to setup a `dev` environment using `composer install/update` or by running:
 
 ```
 $ composer run-script post-install-cmd
 ```

This will generate a `phpcs.xml` file containing settings specific to your local
environment. Make sure to never commit this file.

### Run coding standards checks

#### Run checks manually

The coding standards checks can then be run as follows:

```
# Scan all files for coding standards violations.
$ ./vendor/bin/phpcs

# Scan only a single folder.
$ ./vendor/bin/phpcs docroot/modules/custom/mymodule
```

#### Run checks automatically when pushing

@todo


### Customize configuration

@todo

For more information on configuring the ruleset see [Annotated ruleset](http://pear.php.net/manual/en/package.php.php-codesniffer.annotated-ruleset.php).


## FAQ

### Should I commit the contrib modules I download

Composer recommends **no**. They provide [argumentation against but also 
workrounds if a project decides to do it anyway](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md).

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
            "Patch description": "URL to patch"
        }
    }
}
```

## Notes

This template is based on: https://github.com/pfrenssen/drupal-project, but without usage of Phing and with Triquanta specific thingies.