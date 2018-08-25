# Configuration

## Error Detail

* `debug` option in `config/app.php`.
* `APP_DEBUG` in `.env`.

## Log Storage

* Out of the box, Laravel supports writing log information to `single` files, `daily` files, the `syslog`, and the `errorlog`.

## Maximum Daily Log Files

`'log_max_files' => 30`

## Log Severity Levels

* In production environment, you may wish to configure the minimum severity that should be logged by adding the `log_level` option to `app.php` configuration file.
* For example, a default `log_level` of `error` will log **error, critical, alert**, and **emergency** messages

`'log_level' => env('APP_LOG_LEVEL', 'error'),`

*Monolog recognizes the following severity levels - from least severe to most severe:  debug, info, notice, warning, error, critical, alert, emergency.*

## Custom Monolog Configuration

* To have complete control over how Monolog is configured use `configureMonologUsing` method.
* Place a call to this method in `bootstrap/app.php` file right before the `$app` variable is returned:

```php
$app->configureMonologUsing(function ($monolog) {
    $monolog->pushHandler(...);
});

return $app;
```

## Customizing The Channel Name

* By default, Monolog is instantiated with name that matches the current environment, such as `production` or `local`.
* To change add: `'log_channel' => env('APP_LOG_CHANNEL', 'my-app-name'),`

# The Exception Handler

## The Report Method

* All exceptions are handled by the `App\Exceptions\Handler` class.
* This class contains two methods: `report` and `render`.
* By default, the `report` method passes the exception to the base class where the exception is logged.
* To report different types of exceptions in different ways, use the PHP `instanceof` comparison operator

```php
/**
 * Report or log an exception.
 *
 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
 *
 * @param  \Exception  $exception
 * @return void
 */
public function report(Exception $exception)
{
    if ($exception instanceof CustomException) {
        //
    }

    return parent::report($exception);
}
```

### The `report` Helper

* Sometimes there is need to report an exception but continue handling the current request.
* The `report` helper function allows to quickly report an exception using exception handler's `report` method without rendering an error page.

```php
public function isValid($value)
{
    try {
        // Validate the value...
    } catch (Exception $e) {
        report($e);

        return false;
    }
}
```

### Ignoring Exceptions By Type

* The `$dontReport` property of the exception handler contains an array of exception types that will not be logged.
* For example, exceptions resulting from 404 errors, as well as several other types of errors, are not written to log files.

```php
/**
 * A list of the exception types that should not be reported.
 *
 * @var array
 */
protected $dontReport = [
    \Illuminate\Auth\AuthenticationException::class,
    \Illuminate\Auth\Access\AuthorizationException::class,
    \Symfony\Component\HttpKernel\Exception\HttpException::class,
    \Illuminate\Database\Eloquent\ModelNotFoundException::class,
    \Illuminate\Validation\ValidationException::class,
];
```

## The Render Method

* The `render` method is responsible for converting a given exception into an HTTP response that should be sent back to the browser.

```php
/**
 * Render an exception into an HTTP response.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Exception  $exception
 * @return \Illuminate\Http\Response
 */
public function render($request, Exception $exception)
{
    if ($exception instanceof CustomException) {
        return response()->view('errors.custom', [], 500);
    }

    return parent::render($request, $exception);
}
```

## Reportable & Renderable Exceptions

* Instead of type-checking exceptions in the exception handler's `report` and `render` methods, define `report` and `render` methods directly on custom exception.
* When these methods exist, they will be called automatically by the framework.

```php
<?php

namespace App\Exceptions;

use Exception;

class RenderException extends Exception
{
    /**
     * Report the exception.
     *
     * @return void
     */
    public function report()
    {
        //
    }

    /**
     * Render the exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request
     * @return \Illuminate\Http\Response
     */
    public function render($request)
    {
        return response(...);
    }
}
```

# HTTP Exceptions

* In order to generate a exception from anywhere in app, use the `abort` helper:

```php
abort(404);
```

* The `abort` helper will immediately raise an exception which will be rendered by the exception handler.
* Optional text:
```php
abort(403, 'Unauthorized action.');
```

## Custom HTTP Error Pages

* For example to customize the error page for 404 HTTP status codes, create a `resources/views/errors/404.blade.php`.
* This file will be served on all 404 errors generated by app.
* The views within this directory should be named to match the HTTP status code they correspond to.
* The `HttpException` instance raised by the `abort` function will be passed to the view as an `$exception` variable:

```blade
<h2>{{ $exception->getMessage() }}</h2>
```

# Logging

* Write information to the logs using the `Log` facade.

```php
<?php

namespace App\Http\Controllers;

use App\User;
use Illuminate\Support\Facades\Log;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        Log::info('Showing user profile for user: '.$id);

        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

```
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

## Contextual Information

* An array of contextual data may also be passed to the log methods:

```php
Log::info('User failed to login.', ['id' => $user->id]);
```

## Accessing The Underlying Monolog Instance

* Monolog has a variety of additional handlers you may use for logging. If needed, you may access the underlying Monolog instance being used by Laravel:

```
$monolog = Log::getMonolog();
```