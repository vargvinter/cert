# PSR-7 Requests

* Install:
    * `composer require symfony/psr-http-message-bridge`
    * `composer require zendframework/zend-diactoros`
* Obtain a PSR-7 request:
```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    //
});
```

* Returned PSR-7 response instance from a route or controller will be converted into Laravel response.

# Request Path and Method

## Retrieving The Request Path

`$uri = $request->path();`

## Retrieving The Request URL

```php
// Without Query String...
$url = $request->url();

// With Query String...
$url = $request->fullUrl();
```

## Retrieving The Request Method

```php
$method = $request->method();

if ($request->isMethod('post')) {
    //
}
```

# Retrieving Input

## Retrieving All Input Data

`$input = $request->all();`

## Retrieving An Input Value

* `$name = $request->input('name');`
* Retrieve default value if requested input is not present on the request:
`$name = $request->input('name', 'Sally');`
* Working with forms that contain array inputs use "dot" notation:
```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

## Retrieving Input From The Query String

* `$name = $request->query('name');`

* Retrieve default: 
`$name = $request->query('name', 'Helen');`

* Retrieve all:
`$query = $request->query();`

## Retrieving Input Via Dynamic Properties

When using dynamic properties, Laravel will first look for the parameter's value in the request payload. If it is not present, Laravel will search for the field in the route parameters.

`$name = $request->name;`

## Retrieving JSON Input Values

* `Content-Type` header of the request must be set to `application/json`,
* `$name = $request->input('user.name');`

## Retrieving A Portion Of The Input Data

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

## Determining If An Input Value Is Present

* Is present: 

 ```php
if ($request->has('name')) {
    //
}
```

* All must be present:
```php
if ($request->has(['name', 'email'])) {
    //
}
```

* Is present and is not empty:

```php
if ($request->filled('name')) {
    //
}
```

## Old Input

### Flashing Input To The Session

```php
$request->flash();

$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

### Flashing Input Then Redirecting

```php
return redirect('form')->withInput();

return redirect('form')->withInput(
    $request->except('password')
);
```

### Retrieving Old Input

`$username = $request->old('username');`

`<input type="text" name="username" value="{{ old('username') }}">`

## Cookies

### Retrieving Cookies From Requests

```php
$value = $request->cookie('name');

$value = Cookie::get('name');
```

### Attaching Cookies To Responses

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

### Generating Cookie Instances

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

# Files

## Retrieving Uploaded Files

```php
$file = $request->file('photo');

// OR

$file = $request->photo;
```

Determine if file is present:
```php
if ($request->hasFile('photo')) {
    //
}
```

## Validating Successful Uploads

```php
if ($request->file('photo')->isValid()) {
    //
}
```

## File Paths & Extensions

```php
$path = $request->photo->path();

$extension = $request->photo->extension()
```

## Storing Uploaded Files

* Filename will be random:

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

* Set filename:

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

# Configuring Trusted Proxies

* Use the `App\Http\Middleware\TrustProxies` middleware to quickly customize the load balancers or proxies that should be trusted by application,

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Request;
use Fideloper\Proxy\TrustProxies as Middleware;

class TrustProxies extends Middleware
{
    /**
     * The trusted proxies for this application.
     *
     * @var array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    /**
     * The current proxy header mappings.
     *
     * @var array
     */
    protected $headers = [
        Request::HEADER_FORWARDED => 'FORWARDED',
        Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
        Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
        Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
        Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
    ];
}
```

## Trusting All Proxies

```php
/**
 * The trusted proxies for this application.
 *
 * @var array
 */
protected $proxies = '**';
```

