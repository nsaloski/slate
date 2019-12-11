# Introduction

Welcome to the Caymland Developer Documentation. This documentation will go over how to build a Caymland Plugin that extends the features of Caymland, how to build custom themes, and and how to integrate applications outside of Caymland using its REST API. 

## Submitting Code to Caymland

Development is open and available to any member of the Caymland community. All fixes and improvements are done through pull requests to the code on [GitHub](https://github.com/caymland/caymland). This code is open source and publicly available. 

Read more about the PR process on the Caymland [Governance](https://www.caymland.org/about/governance) page.

The code should try to follow [Symfony's Coding Standards](http://symfony.com/doc/current/contributing/code/standards.html)

## Symfony

Caymland is built on [Symfony](http://symfony.com), the popular PHP framework (Caymland 1.0.x uses Symfony 2.5; Caymland 2.0.x uses Symfony 2.8). This document will go over the basics but most of [their documentation](http://symfony.com/doc/2.8/book/index.html) applies to Caymland as well which can be used to obtain more advanced Symfony functionality.
  
There are some structural differences between Caymland and standard Symfony. Below is a list of where you will find some of standard Symfony locations in Caymland:
 
 Symfony | Caymland
 ------- | -------
 src/ | app/bundles (Caymland core) or plugins/ (Caymland plugins)
 web/ | /
 AcmeBundle/Resources/config | AcmeBundle/Config
 AcmeBundle/Resources/views | AcmeBundle/Views
 AcmeBundle/Resources/public | AcmeBundle/Assets
 AcmeBundle/Resources/translations/domain.en_US.ini | AcmeBundle/Translations/en_US/domain.ini
 
 Most of Symfony's standard locations, such as the Resources/views and Resources/translations directories, should still function with Caymland. However, it may be required to handle service registration, etc with native Symfony processes if not using the Caymland methods defined in this document. 
 
## Development Environment

### Setup
It is assumed that the system already has [composer](https://getcomposer.org) and [git](http://git-scm.com) installed and configured.

To setup the developer environment, simply fork and clone the source from [GitHub](https://github.com/caymland/caymland). Then Run `composer install` on the source. 

Open your browser and complete the installation through the Caymland installer.

### Environments

There are three environments in Caymland: prod, dev, and test.

**prod** is used when accessing the site through index.php.

**dev** is used when accessing the site through index_dev.php. Using Caymland in the dev environment will activate Symfony's profiler toolbar, has more strict error handling, will display information about exceptions, and will not cache as much (see below). Note that steps should be taken to ensure index_dev.php is not accessible to the public as it could potentially reveal sensitive information. It is restricted to localhost by default. However, there are two ways to allow access to index_dev.php from a non-localhost. The first option is to set a header from the web-server with the IP addresses assigned to `CAYMLAND_DEV_HOSTS`. The second and easier option is to add an array to your installation's `app/config/local.php` file as `'dev_hosts' = ['123.123.123.123', '124.124.124.124'],` then clear the [cache](#cache).

**test** is used mainly for PHP Unit Tests.

### Cache

Symfony makes heavy use of a filesystem cache. Frequently clearing this cache will be required when developing for Caymland. By default, the cache is located in app/cache/ENV where ENV is the environment currently accessed (i.e. dev or prod). To rebuild the cache, the ENV can just be deleted or run the Symfony command `php app/console cache:clear --env=ENV` If a specific environment is not passed to the command via `--env=ENV`, the dev environment will be used by default.
 
 In the dev environment, translations, views, and assets are not cached. However, changes to these files will require the cache to be cleared for them to take effect in the prod environment. Changes to Caymland config files, Symfony config files, etc will require the cache to be cleared regardless of environment.
 
 <aside class="notice">
 The typical rule of thumb is, if Caymland is not acting as you expect after making changes, try clearing your cache.
 </aside>
 
 <aside class="warning">
 If you get class could not be found or cannot redeclare class errors when using the cache:clear command, manually delete the app/cache/ENV folder then run the command and/or browse to the site to rebuild.
 </aside>
 
