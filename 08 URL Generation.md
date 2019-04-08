# URL Generation

## Generating Basic URLs

```php
$post = App\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

## Accessing The Current URL

* If no path is provided to the `url` helper, a `Illuminate\Routing\UrlGenerator` instance is returned.
* Access information about the current URL.

```php
// Get the current URL without the query string...
echo url()->current();

// Get the current URL including the query string...
echo url()->full();

// Get the full URL for the previous request...
echo url()->previous();
```

# Named Routes

```php
Route::get('/post/{post}', function () {
    //
})->name('post.show');
```

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

* Eloquent models may be also passed.
* Primary ID wil be extracted.

```php
echo route('post.show', ['post' => $post]);
``` 

# Controller Actions

* The `action` function generates a URL for the given controller action.

```php
$url = action('HomeController@index');

$url = action('UserController@profile', ['id' => 1]);
```

# Default Values

* Specify request-wide default values for certain URL parameters.
* For example, many of routes define a `{locale}` parameter:

```php
Route::get('/{locale}/posts', function () {
    //
})->name('post.index');
```

* Use the `URL::defaults` method to define a default value for this parameter that will always be applied during the current request.
* Call this method from a route middleware:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\URL;

class SetDefaultLocaleForUrls
{
    public function handle($request, Closure $next)
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```
