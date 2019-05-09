# Configuration

The cache configuration is located at `config/cache.php`. 
 
## Drivers
* Database,
* Memcached,
* Redis
* File
* Apc
* Array

## Driver Prerequisites

### Database

```php
Schema::create('cache', function ($table) {
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});
```

OR

`php artisan cache:table` to generate migration with proper schema.

### Memcached

* `Memcached PECL` package must be installed
* `config/cache.php`:
 ```php
'memcached' => [
    [
        'host' => '127.0.0.1',
        'port' => 11211,
        'weight' => 100
    ],
],
```

### Redis

* `predis/predis` package

# Cache Usage

## Obtaining A Cache Instance

* `Illuminate\Contracts\Cache\Factory` and `Illuminate\Contracts\Cache\Repository` contracts provide access to Laravel's cache services.
* The `Factory` contract provides access to all cache drivers defined for your application. 
* The `Repository` contract is typically an implementation of the default cache driver.
* Typically use `Cache` facade.

### Accessing Multiple Cache Stores

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 10);
```

## Retrieving Items From The Cache

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

The result of the Closure will be returned if the specified item does not exist in the cache. Passing a Closure allows you to defer the retrieval of default values from a database or other external service.

```php
$value = Cache::get('key', function () {
    return DB::table(...)->get();
});
```

### Checking For Item Existence

```php
if (Cache::has('key')) {
    //
}
```

### Incrementing / Decrementing Values

The `increment` and `decrement` methods may be used to adjust the value of integer items in the cache.

```php
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

### Retrieve & Store

If data don't exist, retrieve it from the database and add it to the cache.

```php
$value = Cache::remember('users', $minutes, function () {
    return DB::table('users')->get();
});
```

```php
$value = Cache::rememberForever('users', function() {
    return DB::table('users')->get();
});
```

### Retrieve & Delete

```php
$value = Cache::pull('key');
```

## Storing Items In The Cache

```php
Cache::put('key', 'value', $minutes);
```

### Store If Not Present

```php
Cache::add('key', 'value', $minutes);
```

The method will return `true` if the item is actually added to the cache. Otherwise, the method will return `false`.

### Storing Items Forever

```php
Cache::forever('key', 'value');
```

## Removing Items From The Cache

Single value:
```php
Cache::forget('key');
```

All the cache:
```php
Cache::flush();
```

**Flushing the cache does not respect the cache prefix and will remove all entries from the cache. Consider this carefully when clearing a cache which is shared by other applications.**

# Cache Tags

Cache tags are not supported when using the `file` or `database` cache drivers. 

Cache tags allow to tag related items in the cache and then flush all cached values that have been assigned a given tag.

## Storing Tagged Cache Items

```php
Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);
```

## Accessing Tagged Cache Items

```php
$john = Cache::tags(['people', 'artists'])->get('John');

$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

## Removing Tagged Cache Items

You may flush all items that are assigned a tag or list of tags.

```php
Cache::tags(['people', 'authors'])->flush();

Cache::tags('authors')->flush();
```

# Creating Custom Drivers

To create our custom cache driver, we first need to implement the  `Illuminate\Contracts\Cache\Store` contract. 

```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys);
    public function put($key, $value, $minutes) {}
    public function putMany(array $values, $minutes);
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

Once our implementation is complete, we can finish our custom driver registration.

To register the custom cache driver with Laravel, we will use the extend method on the `Cache` facade. The call to `Cache::extend` could be done in the `boot` method of the default  `App\Providers\AppServiceProvider`.

```php
Cache::extend('mongo', function ($app) {
    return Cache::repository(new MongoStore);
});
```
