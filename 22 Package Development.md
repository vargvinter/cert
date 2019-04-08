# Discovery

* Define the provider in the `extra` section of your package's `composer.json` file.
* In addition to service providers, you may also list any facades you would like to be registered:

```
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```

* Disable package discovery

```
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

* disable package discovery for all packages.

```
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

# Service Providers

# Resources

## Configuration

Allow users to override your package's config.

```php
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
    ]);
}
```

### Default Package Configuration

```php
public function register()
{
    $this->mergeConfigFrom(
        __DIR__.'/path/to/config/courier.php', 'courier'
    );
}
```

## Routes

If your package contains routes, you may load them using the loadRoutesFrom method.

```php
public function boot()
{
    $this->loadRoutesFrom(__DIR__.'/routes.php');
}
```

## Migrations

```php
public function boot()
{
    $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
}
```

Once your package's migrations have been registered, they will automatically be run when the  php artisan migrate command is executed.

## Translations

```php
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
}
```

Package translations are referenced using the package::file.line syntax convention. So, you may load the courier package's welcome line from the messages file like so:

`echo trans('courier::messages.welcome');`

### Publishing Translations

```php
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

    $this->publishes([
        __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
    ]);
}
```

## Views

```php
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
}
```

Package views are referenced using the package::view syntax convention.

```php
Route::get('admin', function () {
    return view('courier::admin');
});
```

### Overriding Package Views

* Laravel actually registers two locations for your views: the application's resources/views/vendor directory and the directory you specify.
* Laravel will first check if a custom version of the view has been provided by the developer in resources/views/vendor/courier.
* Then, if the view has not been customized, Laravel will search the package view directory you specified in your call to  loadViewsFrom.

### Publishing Views

```php
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

    $this->publishes([
        __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
    ]);
}
```

# Commands

```php
public function boot()
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            FooCommand::class,
            BarCommand::class,
        ]);
    }
}
```

# Public Assets

* package may have assets such as JavaScript, CSS, and images.
* To publish these assets to the application's public directory, use the service provider's publishes method.

```php
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/assets' => public_path('vendor/courier'),
    ], 'public');
}
```

# Publishing File Groups

You may want to publish groups of package assets and resources separately. For instance, you might want to allow your users to publish your package's configuration files without being forced to publish your package's assets. You may do this by "tagging" them when calling the  publishes method from a package's service provider.

```php
public function boot()
{
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'config');

    $this->publishes([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'migrations');
}
```

`php artisan vendor:publish --tag=config`

