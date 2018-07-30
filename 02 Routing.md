# HTTP Verbs

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

```php
Route::match(['get', 'post'], '/', function () {
    //
});

Route::any('foo', function () {
    //
});
```

# Redirects

```php
return Route::redirect('/somewhere', 301);
```

```php
return redirect('/somewhere', 301);
```

# Route Parameters

## Required Parameters

```php
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

## Optional Parameters

```php
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

## Regular Expression Constraints

```php
Route::get('user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

## Global Constraints

* Define patterns in the `boot` method of `RouteServiceProvider`

```php
public function boot()
{
    Route::pattern('id', '[0-9]+');

    parent::boot();
}
```

* Then:

```php
Route::get('user/{id}', function ($id) {
    // Only executed if {id} is numeric...
});
```

# Named Routes

## Declaration

```php
Route::get('user/profile', function () {
    //
})->name('profile');
```
```php
Route::get('user/profile', 'UserController@showProfile')->name('profile');
```

## Generating URLs To Named Routes

* Without params:

```php
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```

* With params:

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

## Inspecting The Current Route

```php
/**
 * Handle an incoming request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```

# Route Groups

* Allow to share route attributes, such as middleware or namespaces, across a large number of routes without needing to define those attributes on each individual route.

## Middleware

* Assign middleware to all routes within a group:

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second Middleware
    });

    Route::get('user/profile', function () {
        // Uses first & second Middleware
    });
});
```

## Namespaces

```php
Route::namespace('Admin')->group(function () {
    // Controllers Within The "App\Http\Controllers\Admin" Namespace
});
```

## Sub-Domain Routing

```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

## Route Prefixes

```php
Route::prefix('admin')->group(function () {
    Route::get('users', function () {
        // Matches The "/admin/users" URL
    });
});
```

## Route Name Prefixes

```php
Route::name('admin.')->group(function () {
    Route::get('users', function () {
        // Route assigned name "admin.users"...
    });
});
```

# Route Model Binding

## Implicit Binding

```php
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;
});
```

### Customizing The Key Name

* To use model binding with column other than `id` override the `getRouteKeyName` method on the Eloquent model:

```php
public function getRouteKeyName()
{
    return 'slug';
}
```

## Explicit Binding

* Use the router's `model` method to specify the class for a given parameter.
* Define explicit model bindings in the `boot` method of the `RouteServiceProvider` class:

```php
public function boot()
{
    parent::boot();

    Route::model('user', App\User::class);
}
```

* And define a route that contains a `{user}` parameter:

```php
Route::get('profile/{user}', function ($user) {
    //
});
```

## Customizing The Resolution Logic

* Use the `Route::bind` method.
* The `Closure` passed to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

```php
public function boot()
{
    parent::boot();

    Route::bind('user', function ($value) {
        return App\User::where('name', $value)->first() ?? abort(404);
    });
}
```