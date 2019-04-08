# Defining

* Command will place a new `CheckAge` class within `app/Http/Middleware` directory:

```php
php artisan make:middleware CheckAge
```

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckAge
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }

        return $next($request);
    }
}
```

## Before & After Middleware

* Before middleware: 

```php
<?php

namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // Perform action

        return $next($request);
    }
}
```

* After middleware:

```php
<?php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```

# Registering Middleware

## Global middleware

* To run middleware during every HTTP request, add the middleware class in the `$middleware` property of `app/Http/Kernel.php` class.

## Assigning Middleware To Routes

* Assign the middleware a key in `app/Http/Kernel.php` to `$routeMiddleware` property.

```php
protected $routeMiddleware = [
    'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
    // other
];
```

* Use one middleware:

```php
Route::get('admin/profile', function () {
    //
})->middleware('auth');
```

* Use multiple middlewares:

```php
Route::get('/', function () {
    //
})->middleware('first', 'second');
```

* Use with fully qualified class name:

```php
use App\Http\Middleware\CheckAge;

Route::get('admin/profile', function () {
    //
})->middleware(CheckAge::class);
```

## Middleware Groups

* Group several middleware under a single key to make them easier to assign to routes.
* Out of the box, `web` and `api` middleware groups contain common middleware to apply to web UI and API routes.

```php
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:60,1',
        'auth:api',
    ],
];
```

* Assign:

```php
Route::get('/', function () {
    //
})->middleware('web');

Route::group(['middleware' => ['web']], function () {
    //
});
```

# Middleware Parameters

* Additional middleware parameters will be passed to the middleware after the `$next` argument.

```php
public function handle($request, Closure $next, $role)
{
    if (! $request->user()->hasRole($role)) {
        // Redirect...
    }

    return $next($request);
}
```

* Specify middleware parameters when defining the route by separating the middleware name and parameters with a `:`. Multiple parameters should be delimited by commas:

```php
Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
```

# Terminable middlewares

* So some work after the HTTP response has been sent to the browser.
* For example, the "session" middleware writes the session data to storage after the response has been sent to the browser.
* Define `terminate` method in middleware class.

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;

class StartSession
{
    public function handle($request, Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // Store the session data...
    }
}
```

