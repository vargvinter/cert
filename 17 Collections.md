# Creating / Extending Collections

## Creating collections

```php
$collection = collect([1, 2, 3]);
```

## Extending Collections

```php
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function ($value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

Typically, you should declare collection macros in a service provider.

# Higher Order Messages

* They are short-cuts for performing common actions on collections.
* The collection methods that provide higher order messages are: average, avg, contains, each, every, filter, first, flatMap, map,  partition, reject, sortBy, sortByDesc, sum, and unique.
* Each higher order message can be accessed as a dynamic property on a collection instance.

```php
$users = User::where('votes', '>', 500)->get();

$users->each->markAsVip();
```

```php
$users = User::where('group', 'Development')->get();

return $users->sum->votes;
```

