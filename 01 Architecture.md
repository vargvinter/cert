# Request Lifecycle
* `public/index.php` - entry point for all requests. All requests are directed to this file by web server.
* `index.php` - loads the Composer generated autoloader definition and then retrieves an instance of the Laravel application from `bootstrap/app.php` script.
* The incoming request is sent to either the HTTP kernel or the console kernel, depending on the type of request that is entering the application. let's just focus on the HTTP kernel, which is located in `app/Http/Kernel.php`.
* The HTTP kernel extends the `Illuminate\Foundation\Http\Kernel` class, which defines an array of `bootstrappers` that will be run before the request is executed. Some of bootstrappers' tasks:
    * configure error handling, 
    * configure logging, 
    * detect the application environment, 
    * other.
* The HTTP kernel also defines a list of HTTP middleware that all requests must pass through before being handled by the application. Like:
    * reading and writing the HTTP session, 
    * determining if the application is in maintenance mode, 
    * verifying the CSRF token, 
    * and more.
* Loading the service providers from `config/app.php`.
* First, the `register` method will be called on all providers, 
* Once all providers have been registered, the `boot` method will be called.
* Dispatch request - then the `Request` will be handed off to the router for dispatching. The router will dispatch the request to a route or controller, as well as run any route specific middleware.

# Service Container Binding and Resolution

There is no need to bind classes into the container if they do not depend on any interfaces. The container does not need to be instructed on how to build these objects, since it can automatically resolve these objects using **reflection**.

## Simple Bindings

* Within a service provider, you always have access to the container via the `$this->app` property

```php
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```

## Binding A Singleton

* The `singleton` method binds a class or interface into the container that should only be resolved one time. Once a singleton binding is resolved, the same object instance will be returned on subsequent calls into the container.

```php
$this->app->singleton('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```

## Binding Instances

* You may bind an existing object instance into the container using the `instance` method. The given instance will always be returned on subsequent calls into the container:
 ```php
 $api = new HelpSpot\API(new HttpClient);
 
 $this->app->instance('HelpSpot\API', $api);
 ```
 
## Binding Primitives

Sometimes you may have a class that receives some injected classes, but also needs an injected primitive value such as an integer. You may easily use contextual binding to inject any value your class may need:

```php
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);
```
 
## Binding Interfaces To Implementations

```php
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);
```

## Contextual Binding

Two controllers may depend on different implementations of the `Illuminate\Contracts\Filesystem\Filesystem` contract

```php
use Illuminate\Support\Facades\Storage;
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;

$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when(VideoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

## Tagging

```php
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```

## Extending Bindings

The `extend` method allows the modification of resolved services. For example, when a service is resolved, run additional code to decorate or configure the service. The `extend` method accepts a Closure, which should return the modified service, as its only argument:

```php
$this->app->extend(Service::class, function($service) {
    return new DecoratedService($service);
});
```

## Resolving

* The `make` Method
```php
$api = $this->app->make('HelpSpot\API');
```

Without access to the `$app` variable use:

```php
$api = resolve('HelpSpot\API');
```

If some of class' dependencies are not resolvable via the container, inject them by passing them as an associative array into the `makeWith` method:

```php
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);
```

* **Automatic Injection** - you may "type-hint" the dependency in the constructor of a class that is resolved by the container, including controllers, event listeners, queue jobs, middleware, and more.

# Container Events

```php
$this->app->resolving(function ($object, $app) {
    // Called when container resolves object of any type...
});

$this->app->resolving(HelpSpot\API::class, function ($api, $app) {
    // Called when container resolves objects of type "HelpSpot\API"...
});
```

# PSR-11

Laravel's service container implements the PSR-11 interface. Therefore, you may type-hint the PSR-11 container interface to obtain an instance of the Laravel container:

```php
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get('Service');

    //
});
```

**Calling the `get` method will throw an exception if the identifier has not been explicitly bound into the container.**

# Service Providers

## Writing Service Providers

* `php artisan make:provider RiakServiceProvider`

### The Register Method

* Within this method, only bind things into the service container.
* Never register any event listeners, routes, etc. Otherwise, the service may be used before it was loaded by the service provider.

```php
public function register()
{
    $this->app->singleton(Connection::class, function ($app) {
        return new Connection(config('riak'));
    });
}
```

### The Boot Method

* Is called after all other service providers have been registered.
```php
 public function boot()
 {
     view()->composer('view', function () {
         //
     });
 }
```

* Dependencies (dependency injection) may by type-hinted.
```php
public function boot(ResponseFactory $response)
{
    $response->macro('caps', function ($value) {
        //
    });
}
```

## Registering Providers

* `providers` array in `config/app.php` configuration file.
```php
'providers' => [
    // Other Service Providers

    App\Providers\ComposerServiceProvider::class,
],
```

## Deferred Providers

* Use if provider is **only** registering bindings in the service container.
* Defer registration until one of the registered bindings is actually needed.
* Improve the performance.
* The `provides` method should return the service container bindings registered by the provider.
```php
protected $defer = true;

public function register()
{
    $this->app->singleton(Connection::class, function ($app) {
        return new Connection($app['config']['riak']);
    });
}

public function provides()
{
    return [Connection::class];
}
```

# Facades

* Facades provide a "static" interface to classes that are available in the application's service container.
* Laravel facades serve as "static proxies" to underlying classes in the service container.
* All of Laravel's facades are defined in the `Illuminate\Support\Facades` namespace.

## Facades Vs. Dependency Injection

* One of the primary benefits of dependency injection is the ability to swap implementations of the injected class. This is useful during testing since you can inject a mock or stub and assert that various methods were called on the stub.
* It would not be possible to mock or stub a truly static class method. However, since facades use dynamic methods to proxy method calls to objects resolved from the service container, facades can be tested as an injected class instance.  
```php
Route::get('/cache', function () {
    return Cache::get('key');
});
```

```php
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

## Facades Vs. Helper Functions

* Many of helper functions perform the same function as a corresponding facade.
```php
return View::make('profile');

return view('profile');
```
* There is no practical difference between facades and helper functions.
* When using helper functions, they may be tested exactly as the corresponding facade.

```php
Route::get('/cache', function () {
    return cache('key');
});
```

```php
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

## How Facades Work

* Facade is a class that provides access to an object from the container.
* The machinery that makes this work is in the `Facade` class.
* Laravel's facades, and any custom facades, extend the base `Illuminate\Support\Facades\Facade` class.
* The Facade base class makes use of the `__callStatic()` magic-method to defer calls from facade to an object resolved from the container.
```php
class Cache extends Facade
{
    protected static function getFacadeAccessor() { return 'cache'; }
}
```
* The `Cache` facade extends the base `Facade` class and defines the method `getFacadeAccessor()`.
* This method return the name of a service container binding.
* When a user references any static method on the `Cache` facade, Laravel resolves the cache binding from the service container and runs the requested method.

## Real-Time Facades

```php
<?php

namespace App;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     *
     * @param  Publisher  $publisher
     * @return void
     */
    public function publish(Publisher $publisher)
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```
* Injecting a publisher implementation into the method allows us to easily test the method in isolation.
* However, it requires us to always pass a `publisher` instance each time we call the publish method.
* Using real-time facades can maintain the same testability while not being required to explicitly pass a `Publisher` instance. 
* To generate a real-time facade, prefix the namespace of the imported class with `Facades`.
```php
<?php

namespace App;

use Facades\App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     *
     * @return void
     */
    public function publish()
    {
        $this->update(['publishing' => now()]);

        Publisher::publish($this);
    }
}
```

* And test:

```php
<?php

namespace Tests\Feature;

use App\Podcast;
use Tests\TestCase;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A test example.
     *
     * @return void
     */
    public function test_podcast_can_be_published()
    {
        $podcast = factory(Podcast::class)->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```
