# Defining Controllers

```php
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

```php
Route::get('user/{id}', 'UserController@show');
```

# Controllers & Namespaces

* `RouteServiceProvider` loads your route files within a route group that contains the namespace.
* Specify the portion of the class name that comes after the `App\Http\Controllers` portion of the namespace.
* If full controller class is `App\Http\Controllers\Photos\AdminController`, register routes to the controller like so:

```php 
Route::get('foo', 'Photos\AdminController@method');
```

# Single Action Controllers

* Place a single `__invoke` method on the controller.

```php
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class ShowProfile extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function __invoke($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

```php
Route::get('user/{id}', 'ShowProfile');
```

# Controller Middleware

* In `web.php`:

```php
Route::get('profile', 'UserController@show')->middleware('auth');
```

* In `Controller` class:

```php
class UserController extends Controller
{
    /**
     * Instantiate a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->middleware('subscribed')->except('store');
    }
}
```

* In `Controller` class as `Closure`:

```php
$this->middleware(function ($request, $next) {
    // ...

    return $next($request);
});
```

# Resource Controllers

* Generate controller for CRUD routes:

`php artisan make:controller PhotoController --resource`

* Then:

```php
Route::resource('photos', 'PhotoController');
```

* Register many resource controllers at once by passing an array to the resources method:
  
```php
Route::resources([
  'photos' => 'PhotoController',
  'posts' => 'PostController'
]);
```

## Actions Handled By Resource Controller

| Verb | URI | Action |	Route name |
|------|-----|--------|------------|
|GET|/photos|index|photos.index|
|GET|/photos/create|create|photos.create|
POST|/photos|store|photos.store|
GET|/photos/{photo}|show|photos.show|
GET|/photos/{photo}/edit|edit|photos.edit|
PUT/PATCH|/photos/{photo}|update|photos.update|
DELETE|/photos/{photo}|destroy|photos.destroy|

### Specifying The Resource Model

`php artisan make:controller PhotoController --resource --model=Photo`

### Spoofing Form Methods

`{{ method_field('PUT') }}`

## Partial Resource Routes

```php
Route::resource('photo', 'PhotoController', ['only' => [
    'index', 'show'
]]);

Route::resource('photo', 'PhotoController', ['except' => [
    'create', 'store', 'update', 'destroy'
]]);
```

## API Resource Routes

* Exclude routes that present HTML templates such as `create` and `edit`. For convenience, use the `apiResource` method to automatically exclude these two routes:

```php
Route::apiResource('photo', 'PhotoController');
```

* Register many API resource controllers at once

```php
Route::apiResources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

## Naming Resource Routes

* By default, all resource controller actions have a route name. Override them like this:

```php
Route::resource('photo', 'PhotoController', ['names' => [
    'create' => 'photo.build'
]]);
```

## Naming Resource Route Parameters

* By default, `Route::resource` will create the route parameters for resource routes based on the "singularized" version of the resource name. Override:

```php
Route::resource('user', 'AdminUserController', ['parameters' => [
    'user' => 'admin_user'
]]);
```

`/user/{admin_user}`

## Localizing Resource URIs

* By default, `Route::resource` creates resource URIs using English verbs.
* Use the `Route::resourceVerbs` method in the `boot` method of `AppServiceProvider` to override:

```php
<?php

use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

* Then:

`Route::resource('fotos', 'PhotoController')`

will produce:

`/fotos/crear`
 
`/fotos/{foto}/editar`
 
## Supplementing Resource Controllers

* If there must be additional routes in a resource controller beyond the default set of resource routes, define those routes before your call to `Route::resource`.
* Otherwise, the routes defined by the resource method may unintentionally take precedence over supplemental routes

```php
Route::get('photos/popular', 'PhotoController@method');

Route::resource('photos', 'PhotoController');
```

# Dependency Injection & Controllers

## Constructor Injection

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
     * The user repository instance.
     */
    protected $users;

    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
}
```

## Method Injection

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->name;

        //
    }
}
```

* If controller method is also expecting input from a route parameter, list route arguments after other dependencies.

```php
Route::put('user/{id}', 'UserController@update');
```

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the given user.
     *
     * @param  Request  $request
     * @param  string  $id
     * @return Response
     */
    public function update(Request $request, $id)
    {
        //
    }
}
```

# Route Caching

**Closure based routes cannot be cached. To use route caching, you must convert any Closure routes to controller classes.**

`php artisan route:cache`

