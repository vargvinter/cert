# Introduction

* Gates are most applicable to actions which are not related to any model or resource such as viewing an administrator dashboard
* Policies should be used when you wish to authorize an action for a particular model or resource.

# Gates

## Writing gates

* Gates are Closures that determine if a user is authorized to perform a given action and are defined in the `App\Providers\AuthServiceProvider` class using the `Gate` facade.
* Gates always receive a user instance as their first argument, and may optionally receive additional arguments such as a relevant Eloquent model.

```php
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', function ($user, $post) {
        return $user->id == $post->user_id;
    });
}
```

* Gates may also be defined using a `Class@method` style callback string, like controllers:

```php
Gate::define('update-post', 'PostPolicy@update');
```

## Resource Gates

```php
Gate::resource('posts', 'PostPolicy');
```
This is identical to manually defining the following Gate definitions:

```php
Gate::define('posts.view', 'PostPolicy@view');
Gate::define('posts.create', 'PostPolicy@create');
Gate::define('posts.update', 'PostPolicy@update');
Gate::define('posts.delete', 'PostPolicy@delete');
```

* By default, the view, create, update, and delete abilities will be defined
* You may override or add to the default abilities by passing an array as a third argument to the resource method

```php
Gate::resource('posts', 'PostPolicy', [
    'image' => 'updateImage',
    'photo' => 'updatePhoto',
]);
```

* code will create two new Gate definitions - posts.image and  posts.photo

## Authorizing Actions

```php
if (Gate::allows('update-post', $post)) {
    // The current user can update the post...
}

if (Gate::denies('update-post', $post)) {
    // The current user can't update the post...
}
```

* If you would like to determine if a particular user is authorized to perform an action, you may use the forUser method on the Gate facade:

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // The user can update the post...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // The user can't update the post...
}
```

# Creating Policies

## Generating Policies

Policies are classes that organize authorization logic around a particular model or resource. For example, if your application is a blog, you may have a Post model and a corresponding  PostPolicy to authorize user actions such as creating or updating posts.

* `php artisan make:policy PostPolicy`
* The generated policy will be placed in the app/Policies directory.
* If you would like to generate a class with the basic "CRUD" policy methods already included in the class, you may specify a  --model when executing the command: `php artisan make:policy PostPolicy --model=Post`

## Registering Policies

* The AuthServiceProvider included with fresh Laravel applications contains a policies property which maps your Eloquent models to their corresponding policies. 

```php
class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

## Writing Policies

### Policy Methods

* Once the policy has been registered, you may add methods for each action it authorizes:

```php
<?php

namespace App\Policies;

use App\User;
use App\Post;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

### Methods Without Models

Some policy methods only receive the currently authenticated user and not an instance of the model they authorize. This situation is most common when authorizing create actions. For example, if you are creating a blog, you may wish to check if a user is authorized to create any posts at all.

When defining policy methods that will not receive a model instance, such as a create method, it will not receive a model instance. Instead, you should define the method as only expecting the authenticated user:

```php
public function create(User $user)
{
    //
}
```

## Policy Filters

For certain users, you may wish to authorize all actions within a given policy. This feature is most commonly used for authorizing application administrators to perform any action:
                                                                                
```php
public function before($user, $ability)
{
    if ($user->isSuperAdmin()) {
        return true;
    }
}
```

If you would like to deny all authorizations for a user you should return false from the before method. If null is returned, the authorization will fall through to the policy method.

**The before method of a policy class will not be called if the class doesn't contain a method with a name matching the name of the ability being checked.**

## Authorizing Actions Using Policies

### Via The User Model

```php
if ($user->can('update', $post)) {
    //
}
```

If a policy is registered for the given model, the can method will automatically call the appropriate policy and return the boolean result. If no policy is registered for the model, the  can method will attempt to call the Closure based Gate matching the given action name.

#### Actions That Don't Require Models

```php
use App\Post;

if ($user->can('create', Post::class)) {
    // Executes the "create" method on the relevant policy...
}
```

### Via Middleware

```php
use App\Post;

Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');
```

In this example, we're passing the can middleware two arguments. The first is the name of the action we wish to authorize and the second is the route parameter we wish to pass to the policy method. In this case, since we are using implicit model binding, a Post model will be passed to the policy method. If the user is not authorized to perform the given action, a HTTP response with a 403 status code will be generated by the middleware.

#### Actions That Don't Require Models

```php
Route::post('/post', function () {
    // The current user may create posts...
})->middleware('can:create,App\Post');
```

### Via Controller Helpers

In addition to helpful methods provided to the User model, Laravel provides a helpful  authorize method to any of your controllers which extend the  App\Http\Controllers\Controller base class. Like the can method, this method accepts the name of the action you wish to authorize and the relevant model. If the action is not authorized, the authorize method will throw an Illuminate\Auth\Access\AuthorizationException, which the default Laravel exception handler will convert to an HTTP response with a 403 status code:

```php
<?php

namespace App\Http\Controllers;

use App\Post;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * Update the given blog post.
     *
     * @param  Request  $request
     * @param  Post  $post
     * @return Response
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);

        // The current user can update the blog post...
    }
}
```

#### Actions That Don't Require Models

```php
public function create(Request $request)
{
    $this->authorize('create', Post::class);

    // The current user can create blog posts...
}
```

### Via Blade Templates

```blade
@can('update', $post)
    <!-- The Current User Can Update The Post -->
@elsecan('create', App\Post::class)
    <!-- The Current User Can Create New Post -->
@endcan

@cannot('update', $post)
    <!-- The Current User Can't Update The Post -->
@elsecannot('create', App\Post::class)
    <!-- The Current User Can't Create New Post -->
@endcannot
```

#### Actions That Don't Require Models

```blade
@can('create', App\Post::class)
    <!-- The Current User Can Create Posts -->
@endcan

@cannot('create', App\Post::class)
    <!-- The Current User Can't Create Posts -->
@endcannot
```

