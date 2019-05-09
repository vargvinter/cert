# Configuring The Locale

In the `config/app.php` configuration file.

Or on the fly:

```php
Route::get('welcome/{locale}', function ($locale) {
    App::setLocale($locale);

    //
});
```

`'fallback_locale' => 'en'`

## Determining The Current Locale

```php
$locale = App::getLocale();

if (App::isLocale('en')) {
    //
}
```

## Using Translation Strings As Keys

For applications with heavy translation requirements, defining every string with a "short key" can become quickly confusing when referencing them in views.

`resources/lang/es.json`

```
{
    "I love programming.": "Me encanta programar."
}
```

# Retrieving Translation Strings

```php
echo __('messages.welcome');

echo __('I love programming.');
```

```blade
{{ __('messages.welcome') }}

@lang('messages.welcome')
```

## Replacing Parameters In Translation Strings

`'welcome' => 'Welcome, :name',`

```php
echo __('messages.welcome', ['name' => 'dayle']);
```

```
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
```

# Pluralization

`'apples' => 'There is one apple|There are many apples'`

`'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',`

```php
echo trans_choice('messages.apples', 10);
```

With placeholders:

```
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```