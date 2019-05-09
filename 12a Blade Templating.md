# Layout

```blade
<!-- Stored in resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

```blade
<!-- Stored in resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

Contrary to the previous example, this sidebar section ends with @endsection instead of @show. The @endsection directive will only define a section while @show will define and immediately yield the section.

Use `@parent` directive to append (rather than overwriting) content to the layout's sidebar.

So, if you already have a `@section` defined in the master layout, it will be overriden unless you specify `@parent` inside the child layout's @section.

But for `@yield`, it always gets the section from the child layout. That means it always overrides the `@yield` part, even if it has a default defined as `@yield('section', 'Default Content')`.

# Components & Slots

```blade
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

The `{{ $slot }}` variable will contain the content we wish to inject into the component.

```blade
@component('alert')
    <strong>Whoops!</strong> Something went wrong!
@endcomponent
```

## Named slots

```blade
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>

    {{ $slot }}
</div>
```

```blade
@component('alert')
    @slot('title')
        Forbidden
    @endslot

    You are not allowed to access this resource!
@endcomponent
```

### Passing Additional Data To Components

```blade
@component('alert', ['foo' => 'bar'])
    ...
@endcomponent
```

# Displaying data

Blade `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks.

Displaying Unescaped Data:

`Hello, {!! $name !!}.`

Rendering JSON:

```blade
<script>
    var app = @json($array);
</script>
```

# Blade & JavaScript Frameworks

Use the `@` symbol to inform the Blade rendering engine an expression should remain untouched.

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

Display JS variables in a large portion of template:

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

# Control Structures

## If Statements

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Opposite to `@if`

```blade
@unless (Auth::check())
    You are not signed in.
@endunless
```

```blade
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

## Authentication Directives

```blade
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

With guard `@auth('admin')`.

## Section Directives

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

## Switch Statements

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

## Loops

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

Include the condition with the directive declaration in one line

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

### The Loop Variable

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

If you are in a nested loop, you may access the parent loop's $loop variable via the parent property:

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

| Property	| Description |
|---|---|
| $loop->index	| The index of the current loop iteration (starts at 0). |
|$loop->iteration |	The current loop iteration (starts at 1).|
|$loop->remaining	| The iteration remaining in the loop.|
|$loop->count	| The total number of items in the array being iterated.|
|$loop->first	| Whether this is the first iteration through the loop.|
|$loop->last	| Whether this is the last iteration through the loop.|
|$loop->depth	| The nesting level of the current loop.|
|$loop->parent	| When in a nested loop, the parent's loop variable.|

# Comments

```blade
{{-- This comment will not be present in the rendered HTML --}}
```

# PHP

```blade
@php
    //
@endphp
```

# Including Sub-Views

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

With vars:

```blade
@include('view.name', ['some' => 'data'])
```

Include a view that may or may not be present:

```blade
@includeIf('view.name', ['some' => 'data'])
```

`@include` a view depending on a given boolean condition, use the  `@includeWhen` directive.

```blade
@includeWhen($boolean, 'view.name', ['some' => 'data'])
```

Include the first view that exists from a given array of views:

```blade
@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
```

# Rendering Views For Collections

```blade
@each('view.name', $jobs, 'job')
```

* The first argument is the view partial to render.
* The second argument is the array or collection to iterate over.
* third argument is the variable name that will be assigned to the current iteration within the view.
* The key for the current iteration will be available as the key variable within view partial.


If array is empty the view in forth argument will be rendered:

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

Views rendered via `@each` do not inherit the variables from the parent view. If the child view requires these variables, you should use `@foreach` and `@include` instead.

# Stacks

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

# Service Injection

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

# Extending Blade

The following example creates a @datetime($var) directive which formats a given $var, which should be an instance of DateTime:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::directive('datetime', function ($expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

After updating the logic of a Blade directive, you will need to delete all of the cached Blade views. The cached Blade views may be removed using the view:clear Artisan command.

## Custom If Statements

```php
public function boot()
{
    Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

```blade
@env('local')
    // The application is in the local environment...
@elseenv('testing')
    // The application is in the testing environment...
@else
    // The application is not in the local or testing environment...
@endenv
```