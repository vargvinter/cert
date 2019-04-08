# Introduction

Artisan is the command-line interface included with Laravel.

* List all commands `php artisan list`.
* Every command also includes a "help" screen which displays and describes the command's available arguments and options.
```
php artisan help migrate
```

* All Laravel applications include Tinker, a REPL powered by the PsySH package.
* REPL - read-eval-print loop. Easy interactive programming environment.

```
php artisan tinker
```

# Writing Commands

* Commands are typically stored in the `app/Console/Commands` directory.

## Generating Commands

* To create a new command, use the `make:command` Artisan command.

```
php artisan make:command SendEmails
``` 

## Command Structure

After generating your command, you should fill in the `signature` and `description` properties of the class, which will be used when displaying your command on the list screen.
The  `handle` method will be called when your command is executed. You may place your command logic in this method.

## Closure Commands

Closure based commands provide an alternative to defining console commands as classes.

* Within the `commands` method of your `app/Console/Kernel.php` file, Laravel loads the `routes/console.php` file.
* Within this file, you may define all of your Closure based routes using the `Artisan::command` method.
* The command method accepts two arguments: the command signature and a Closure which receives the commands arguments and options.

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
});
```

### Type-Hinting Dependencies

```php
use App\User;
use App\DripEmailer;

Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
    $drip->send(User::find($user));
});
```

### Closure Command Descriptions

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
})->describe('Build the project');
```

# Defining Input Expectations

* Laravel makes it very convenient to define the input you expect from the user using the `signature` property on your commands.
* The `signature` property allows you to define the name, arguments, and options for the command in a single, expressive, route-like syntax.

## Arguments

* All user supplied arguments and options are wrapped in curly braces.

    * Required argument:

    ```
    protected $signature = 'email:send {user}';
    ```

    * Optional argument
    
    ```
    protected $signature = 'email:send {user?}';
    ```
    
    * Optional argument with default value
    
    ```
    protected $signature = 'email:send {user=foo}';
    ```
    
## Options

Options, like arguments, are another form of user input. Options are prefixed by two hyphens `(--)` when they are specified on the command line. There are two types of options: those that receive a value and those that don't. Options that don't receive a value serve as a boolean "switch".

```php
protected $signature = 'email:send {user} {--queue}';
```

In this example, the --queue switch may be specified when calling the Artisan command. If the  --queue switch is passed, the value of the option will be true. Otherwise, the value will be  false.

### Options With Values

If the user must specify a value for an option, suffix the option name with a `=` sign.

```php
protected $signature = 'email:send {user} {--queue=}';
```

You may assign default values to options by specifying the default value after the option name. If no option value is passed by the user, the default value will be used.

```php
protected $signature = 'email:send {user} {--queue=foo}';
```

### Option Shortcuts

To assign a shortcut when defining an option, you may specify it before the option name and use a | delimiter to separate the shortcut from the full option name.

```
email:send {user} {--Q|queue}
```

## Input Arrays

If you would like to define arguments or options to expect array inputs, you may use the `*` character.

```
email:send {user*}
```

For example, the following command will set the value of user to `['foo', 'bar']`:

```
php artisan email:send foo bar
```

When defining an option that expects an array input, each option value passed to the command should be prefixed with the option name.

```
email:send {user} {--id=*}

php artisan email:send --id=1 --id=2
```

## Input Descriptions

You may assign descriptions to input arguments and options by separating the parameter from the description using a colon.

```php
protected $signature = 'email:send
                        {user : The ID of the user}
                        {--queue= : Whether the job should be queued}';
```

# Command I/O

## Retrieving Input

```php
public function handle()
{
    $userId = $this->argument('user');

    //
}
```

If you need to retrieve all of the arguments as an array, call the arguments method:

```php
$arguments = $this->arguments();
```

```php
// Retrieve a specific option...
$queueName = $this->option('queue');

// Retrieve all options...
$options = $this->options();
```

If the argument or option does not exist, `null` will be returned.

## Prompting For Input

The ask method will prompt the user with the given question, accept their input, and then return the user's input back to your command.

```php
public function handle()
{
    $name = $this->ask('What is your name?');
}
```

The `secret` method is similar to ask, but the user's input will not be visible to them as they type in the console. This method is useful when asking for sensitive information such as a password.

```php
$password = $this->secret('What is the password?');
```

### Asking For Confirmation

If you need to ask the user for a simple confirmation, you may use the confirm method. By default, this method will return false. However, if the user enters y or yes in response to the prompt, the method will return true.

```php
if ($this->confirm('Do you wish to continue?')) {
    //
}
```

### Auto-Completion

The anticipate method can be used to provide auto-completion for possible choices.

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

### Multiple Choice Questions

If you need to give the user a predefined set of choices, you may use the choice method. You may set the array index of the default value to be returned if no option is chosen.

```php
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
```

## Writing Output

To send output to the console, use the `line`, `info`, `comment`, `question` and `error` methods. Each of these methods will use appropriate ANSI colors for their purpose.

### Table Layouts

```php
$headers = ['Name', 'Email'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```

### Progress Bars

```php
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

# Registering Commands

Because of the load method call in your console kernel's commands method, all commands within the `app/Console/Commands` directory will automatically be registered with Artisan.
You may also manually register commands by adding its class name to the `$commands` property of your `app/Console/Kernel.php` file.

```php
protected $commands = [
    Commands\SendEmails::class
];
``` 

# Programmatically Executing Commands

Use the `call` method on the `Artisan` facade to accomplish this.

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

Using the `queue` method on the `Artisan` facade, you may even queue Artisan commands so they are processed in the background by your queue workers.

```php
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

You may also specify the connection or queue the Artisan command should be dispatched to:

```php
Artisan::queue('email:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

## Passing Array Values

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--id' => [5, 13]
    ]);
});
```

## Passing Boolean Values

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

## Calling Commands From Other Commands

```php
public function handle()
{
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
}
```

If you would like to call another console command and suppress all of its output, you may use the `callSilent` method. The `callSilent` method has the same signature as the `call` method:

```php
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```

