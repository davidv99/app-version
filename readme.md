# Track the version of your app

[![Build Status](https://travis-ci.org/placetopay-org/app-version.svg?branch=master)](https://travis-ci.org/placetopay-org/app-version)

The `placetopay/app-version` package provides a way to know which version of your app is currently deployed.

It can be used with [Envoyer](https://envoyer.io/) deployment hooks

It can be integrated with [Sentry](https://sentry.io) to help you keep track of Releases and Deploys in the Sentry Dashboard 

## Installation

Install using composer

```bash
composer require placetopay/app-version ^2.0
```

The package will automatically register itself and it should work now on your local environment

If you are using Sentry please follow this steps to configure the deployments and releases to your reports

1. Publish the configuration file

    ```bash
    php artisan vendor:publish --provider="PlacetoPay\AppVersion\VersionServiceProvider"
    ```

2. Set up your environment variables at `config/app-version.php`

    ```php
    return [
        'sentry' => [
            /*
             * The sentry auth token used to authenticate with Sentry Api
             * Create tokens here at account level https://sentry.io/settings/account/api/auth-tokens/
             * Or here at organization level https://sentry.io/settings/your-organization/developer-settings/
             */
            'auth_token' => env('APP_VERSION_SENTRY_AUTH'),

            /*
             * The organization name this project belongs to, you can find out the
             * organization at https://sentry.io/settings/
             */
            'organization' => env('APP_VERSION_SENTRY_ORGANIZATION', 'placetopay'),

            /*
             * The repository name of this project. Repositories are added in sentry as integrations.
             * You must add your (Github|Bitbucket) integration at https://sentry.io/settings/your-organization/integrations/
             * and then add the repositories you want to track commits
             */
            'repository' => env('APP_VERSION_SENTRY_REPOSITORY'),

            /*
             * The name of this project in Sentry Dashboard
             * You can find out the project name at https://sentry.io/settings/your-organization/projects/
             */
            'project' => env('APP_VERSION_SENTRY_PROJECT'),
        ],

        /*
         * The current deployed version, will be read from version file
         * generated by `php artisan app-version:create ...` command
         */
        'version' => \PlacetoPay\AppVersion\VersionFile::readSha(),
    ];
    ```

3. Set up the `config/sentry.php` file with the following settings
    ```php
    return [

        'dsn' => env('SENTRY_LARAVEL_DSN', env('SENTRY_DSN')),

        // capture release as git sha
        'release' => \PlacetoPay\AppVersion\VersionFile::readSha(),

        'breadcrumbs' => [
            // Capture Laravel logs in breadcrumbs
            'logs' => true,

            // Capture SQL queries in breadcrumbs
            'sql_queries' => true,

            // Capture bindings on SQL queries logged in breadcrumbs
            'sql_bindings' => true,

            // Capture queue job information in breadcrumbs
            'queue_info' => true,
        ],

    ];
    ```

## Usage

You can visit `https://yourapp.com/version`

### Envoyer Hooks

Using tools to deploy like [Envoyer](https://envoyer.io) there is no git source available once deployed so using the sha, project and branch available information we create a file containing this information

1. Create a deployment hook in the action "Activate New Release", it is vital that this hook runs **BEFORE running config:cache or optimize commands**

    ```shell
    cd {{ release }}
    php artisan app-version:create --sha={{ sha }} --time={{ time }} --project={{ project }} --branch={{ branch }}
    ```

    This will generate your version file at `storage/app/app-version.json` 

2. If you are integrating with Sentry Releases/Deployments/Issues, Add these hooks so Sentry can track your deployments. It should be run AFTER running the optimization or configuration cache

    ```shell
    cd {{ release }}
    php artisan app-version:create-release
    php artisan app-version:create-deploy
    ``` 
### Know your version from CLI

If you're using tinker you can get the version information with the following commands

To access the version information generated with the step 1 of the usage
```php 
config('app-version.version'); 
```

To access the sha
```php 
PlacetoPay\AppVersion\VersionFile::readSha()
```
