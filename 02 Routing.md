# HTTP Verbs

```
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

```
Route::match(['get', 'post'], '/', function () {
    //
});

Route::any('foo', function () {
    //
});
```

# Redirects

`Route::redirect('/here', '/there', 301);`

# Route Parameters

## Required Parameters

```
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

## Optional Parameters

```
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

## Regular Expression Constraints

```
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

```
public function boot()
{
    Route::pattern('id', '[0-9]+');

    parent::boot();
}
```

* Then:

```
Route::get('user/{id}', function ($id) {
    // Only executed if {id} is numeric...
});
```

