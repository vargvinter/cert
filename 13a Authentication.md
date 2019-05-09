# Manually Authenticating Users

```php
if (Auth::attempt(['email' => $email, 'password' => $password])) {
    // Authentication passed...
    return redirect()->intended('dashboard');
}
```

## Additional conditions

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // The user is active, not suspended, and exists.
}
```

## Accessing Specific Guard Instances

```php
if (Auth::guard('admin')->attempt($credentials)) {
    //
}
```

## Remembering Users

```php
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```

* If you are "remembering" users, you may use the viaRemember method to determine if the user was authenticated using the "remember me" cookie:

```php
if (Auth::viaRemember()) {
    //
}
```

## Other Authentication Methods

### Authenticate A User Instance

The given object must be an implementation of the  `Illuminate\Contracts\Auth\Authenticatable` contract.

```php
Auth::login($user);

// Login and "remember" the given user...
Auth::login($user, true);

Auth::guard('admin')->login($user);
```

### Authenticate A User By ID

```php
Auth::loginUsingId(1);

// Login and "remember" the given user...
Auth::loginUsingId(1, true);
```

### Authenticate A User Once

Helpful when building a stateless API.

```php
if (Auth::once($credentials)) {
    //
}
```

# HTTP Basic Authentication

```php
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic');
```

* By default, the `auth.basic` middleware will use the email column on the user record as the "username".

## Stateless HTTP Basic Authentication

* particularly useful for API authentication
* define a middleware that calls the onceBasic method.

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Support\Facades\Auth;

class AuthenticateOnceWithBasicAuth
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, $next)
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

```php
Route::get('api/user', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic.once');
```

