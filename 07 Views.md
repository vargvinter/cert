# Creating Views

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```
```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});

```

## Determining If A View Exists

```php
use Illuminate\Support\Facades\View;

if (View::exists('emails.customer')) {
    //
}
```

## Creating The First Available View

```php
return view()->first(['custom.admin', 'admin'], $data);
```

# Passing Data To Views

```php
return view('greetings', ['name' => 'Victoria']);
```

```php
return view('greeting')->with('name', 'Victoria');
```

## Sharing Data With All Views

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        View::share('key', 'value');
    }

    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

# View Composers

* View composers are callbacks or class methods that are called when a view is rendered.

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function boot()
    {
        // Using class based composers...
        View::composer(
            'profile', 'App\Http\ViewComposers\ProfileComposer'
        );

        // Using Closure based composers...
        View::composer('dashboard', function ($view) {
            //
        });
    }

    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

* Now that we have registered the composer, the `ProfileComposer@compose` method will be executed each time the `profile` view is being rendered.

```php
<?php

namespace App\Http\ViewComposers;

use Illuminate\View\View;
use App\Repositories\UserRepository;

class ProfileComposer
{
    /**
     * The user repository implementation.
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * Create a new profile composer.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        // Dependencies automatically resolved by service container...
        $this->users = $users;
    }

    /**
     * Bind data to the view.
     *
     * @param  View  $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

## Attaching A Composer To Multiple Views

```php
View::composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```

* The composer method also accepts the `*` character as a wildcard, allowing to attach a composer to all views:

```php
View::composer('*', function ($view) {
    //
});
```

## View Creators

* Similar to view composers.
* Executed immediately after the view is instantiated instead of waiting until the view is about to render.

```php
View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
```