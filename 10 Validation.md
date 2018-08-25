# Form Requests

## Creating Form Requests

* `php artisan make:request StoreBlogPost`.
* Class will be placed in the `app/Http/Requests` directory.

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

```php
/**
 * Store the incoming blog post.
 *
 * @param  StoreBlogPost  $request
 * @return Response
 */
public function store(StoreBlogPost $request)
{
    // The incoming request is valid...
}
```

### Adding After Hooks To Form Requests

* To add an "after" hook to a form request, use the `withValidator` method.
* Method receives the fully constructed validator, allowing to call any of its methods before the validation rules are actually evaluated.

```php
/**
 * Configure the validator instance.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```

## Authorizing Form Requests

* Within this method, check if the authenticated user actually has the authority to update a given resource.
* Since all form requests extend the base Laravel request class, we may use the `user` method to access the currently authenticated user.

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

* Also note the call to the `route` method in the example above. This method grants access to the URI parameters defined on the route being called, such as the `{comment}` parameter in the example below:

```php
Route::post('comment/{comment}');
```

* If the authorize method returns false, a HTTP response with a 403 status code will automatically be returned and controller method will not execute.
* If authorization logic is in another part of app, return true from the `authorize` method.

## Customizing The Error Messages

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
    ];
}
```

# Manually Creating Validators

```php
<?php

namespace App\Http\Controllers;

use Validator;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // Store the blog post...
    }
}
```

* Use the `withErrors` method to flash the error messages to the session.
* When using this method, the `$errors` variable will automatically be shared with views after redirection.
* The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.
  
## Automatic Redirection

* Call the `validate` method on an existing validator instance:

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

## Named Error Bags

* If there are multiple forms on a single page, give a name to the `MessageBag` of errors, which allows to retrieve the error messages for a specific form.

```php
return redirect('register')
            ->withErrors($validator, 'login');
```

* Then:

```blade
{{ $errors->login->first('email') }}
```

## After Validation Hook

* Attach callbacks to be run after validation is completed.
* This allows to easily perform further validation and even add more error messages to the message collection.

```php
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```

# Error Messages

## Retrieving The First Error Message For A Field

```php
$errors = $validator->errors();

echo $errors->first('email');
```

## Retrieving All Error Messages For A Field

```php
foreach ($errors->get('email') as $message) {
    //
}
```

* If validating an array form field, retrieve all of the messages for each of the array elements using the `*` character:

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

## Retrieving All Error Messages For All Fields

```php
foreach ($errors->all() as $message) {
    //
}
```

## Determining If Messages Exist For A Field

```php
if ($errors->has('email')) {
    //
}
```

## Custom Error Messages

* There are several ways to specify custom messages.

```php
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```

* Specifying A Custom Message For A Given Attribute

```php
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```

* Specifying Custom Messages In Language Files

    * Add messages to `custom` array in the `resources/lang/xx/validation.ph`p language file:
    
```php
'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```

* Specifying Custom Attributes In Language Files
    * Specify the custom name in the `attributes` array of `resources/lang/xx/validation.php` language file:

```php
'attributes' => [
    'email' => 'email address',
],
```

# Custom Validation Rules

## Using Rule Objects

* `php artisan make:rule Uppercase`
* Define its behaviour.
```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    /**
     * Determine if the validation rule passes.
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be uppercase.';
        
        // OR 
        
        return trans('validation.uppercase');
    }
}
```

* Use:

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', new Uppercase],
]);
```

## Using Extensions

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Validator;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
            return $value == 'foo';
        });
        
        //OR 
        
        Validator::extend('foo', 'FooValidator@validate');
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

### Defining The Error Message

* Do so using an inline custom message array or by adding an entry in the validation language file.
* Message should be placed in the first level of the array, not within the `custom` array.

```php
"foo" => "Your input was invalid!",

"accepted" => "The :attribute must be accepted.",

// The rest of the validation error messages...
```

* Defining custom place-holder replacements for error messages.

```php
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Validator::extend(...);

    Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
        return str_replace(...);
    });
}
```

## Implicit Extensions

* By default, when an attribute being validated is not present or contains an empty value as defined by the required rule, normal validation rules, including custom extensions, are not run. 
* For a rule to run even when an attribute is empty, the rule must imply that the attribute is required.

```php
Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
    return $value == 'foo';
});
```

