# Configuration

* Configuration file is stored at `config/session.php`.
* Database driver prerequisites:

```php
Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->unsignedInteger('user_id')->nullable();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity');
});
```

```
php artisan session:table

php artisan migrate
```

* Redis driver prerequisites:

    * Install the `predis/predis` package (~1.0) via `Composer`.
    
# Storing Data

```php
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global helper...
session(['key' => 'value']);
```

## Pushing To Array Session Values

* The `push` method is used to push a new value onto a session value that is an array.

```php
$request->session()->push('user.teams', 'developers');
```

# Retrieving Data

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function show(Request $request, $id)
    {
        $value = $request->session()->get('key');

        //
    }
}
```

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

```php
Route::get('home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

## Retrieving All Session Data

```php
$data = $request->session()->all();
```

## Determining If An Item Exists In The Session

```php
if ($request->session()->has('users')) {
    //
}
```

## Retrieving & Deleting An Item

```php
$value = $request->session()->pull('key', 'default');
```

# Deleting Data

```php
// Remove piece of data.
$request->session()->forget('key');

// Remove all session data.
$request->session()->flush();
```

# Flash Data

* Store items in the session only for the next request.
* Data stored in the session using this method will only be available during the subsequent HTTP request, and then will be deleted.
* Flash data is primarily useful for short-lived status messages.

```php
$request->session()->flash('status', 'Task was successful!');
```

* If need to keep flash data around for several requests, use the `reflash`:

```php
$request->session()->reflash();
```

* If only need to keep specific flash data, use the `keep` method:

```php
$request->session()->keep(['username', 'email']);
```

# Custom Drivers

* Custom session driver should implement the `SessionHandlerInterface`.
```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

* `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will almost never need to put anything in this method.
* The `close` method, like the open method, can also usually be disregarded. For most drivers, it is not needed.
* The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
* The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB, Dynamo, etc. Again, you should not perform any serialization - Laravel will have already handled that for you.
* The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
* The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.

## Registering The Driver

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Session::extend('mongo', function ($app) {
            // Return implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

* Then use the `mongo` driver in `config/session.php` configuration file.