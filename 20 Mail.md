# Drivers / Configuration

Laravel provides a clean, simple API over the popular SwiftMailer library with drivers for SMTP, Mailgun, SparkPost, Amazon SES, PHP's mail function, and sendmail.

All of the API drivers require the Guzzle HTTP library.

`composer require guzzlehttp/guzzle`

## Drivers

SMTP, Mailgun, SparkPost, SES, mail, sendmail.

# Generating Mailables

* Classes are stored in the app/Mail directory.
* `php artisan make:mail OrderShipped`

# Writing Mail

## Configuring The Sender

This way:

```php
public function build()
{
    return $this->from('example@example.com')
                ->view('emails.orders.shipped');
}
```

Or global:

`config/mail.php` configuration file.

`'from' => ['address' => 'example@example.com', 'name' => 'App Name'],`

## Configuring The View

```php
public function build()
{
    return $this->view('emails.orders.shipped');
}
```

### Plain Text Emails

```php
public function build()
{
    return $this->view('emails.orders.shipped')
                ->text('emails.orders.shipped_plain');
}
```

## View Data

### Via Public Properties

```php
<?php

namespace App\Mail;

use App\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    public $order;

    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    public function build()
    {
        return $this->view('emails.orders.shipped');
    }
}
```

### Via The `with` Method

```php
public function build()
{
    return $this->view('emails.orders.shipped')
                ->with([
                    'orderName' => $this->order->name,
                    'orderPrice' => $this->order->price,
                ]);
}
```

## Attachments

```php
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attach('/path/to/file');
}
```

Add more data:

```php
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attach('/path/to/file', [
                    'as' => 'name.pdf',
                    'mime' => 'application/pdf',
                ]);
}
```

### Raw Data Attachments

For example, you might use this method if you have generated a PDF in memory and want to attach it to the email without writing it to disk.
The attachData method accepts the raw data bytes as its first argument, the name of the file as its second argument, and an array of options as its third argument.

```php
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }
```

## Inline Attachments

* use the embed method on the $message variable within your email template.
* Laravel automatically makes the $message variable available to all of your email templates
* **`$message` variable is not available in markdown messages.**

```blade
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToFile) }}">
</body>
```

### Embedding Raw Data Attachments

If you already have a raw data string you wish to embed into an email template, you may use the embedData method on the $message variable:

```blade
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, $name) }}">
</body>
```

## Customizing The SwiftMailer Message

```php
public function build()
{
    $this->view('emails.orders.shipped');

    $this->withSwiftMessage(function ($message) {
        $message->getHeaders()
                ->addTextHeader('Custom-Header', 'HeaderValue');
    });
}
```

# Sending Mail

* To send a message, use the to method on the Mail facade.
* The to method accepts an email address, a user instance, or a collection of users.
* If you pass an object or collection of objects, the mailer will automatically use their email and name properties when setting the email recipients.

```php
<?php

namespace App\Http\Controllers;

use App\Order;
use App\Mail\OrderShipped;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;
use App\Http\Controllers\Controller;

class OrderController extends Controller
{
    /**
     * Ship the given order.
     *
     * @param  Request  $request
     * @param  int  $orderId
     * @return Response
     */
    public function ship(Request $request, $orderId)
    {
        $order = Order::findOrFail($orderId);

        // Ship order...

        Mail::to($request->user())->send(new OrderShipped($order));
    }
}
```

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

## Queueing Mail

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

### Delayed Message Queueing

```php
$when = now()->addMinutes(10);

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later($when, new OrderShipped($order));
```

### Pushing To Specific Queues

Since all mailable classes generated using the make:mail command make use of the  Illuminate\Bus\Queueable trait, you may call the onQueue and onConnection methods.

```php
$message = (new OrderShipped($order))
                ->onConnection('sqs')
                ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

### Queueing By Default

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    //
}
```

# Markdown Mailables

## Generating Markdown Mailables

`php artisan make:mail OrderShipped --markdown=emails.orders.shipped`

Then, when configuring the mailable within its build method, call the markdown method instead of the view method.

```php
public function build()
{
    return $this->from('example@example.com')
                ->markdown('emails.orders.shipped');
}
```

## Writing Markdown Messages

```blade
@component('mail::message')
# Order Shipped

Your order has been shipped!

@component('mail::button', ['url' => $url])
View Order
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

### Button Component

```blade
@component('mail::button', ['url' => $url, 'color' => 'green'])
View Order
@endcomponent
```

### Panel Component

```blade
@component('mail::panel')
This is the panel content.
@endcomponent
```

### Table Component

```blade
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```

## Customizing The Components

`php artisan vendor:publish --tag=laravel-mail`

This command will publish the Markdown mail components to the  resources/views/vendor/mail directory.

### Customizing The CSS

After exporting the components, the resources/views/vendor/mail/html/themes directory will contain a default.css file.

# Local Development

## Log Driver

The log mail driver will write all email messages to your log files for inspection.

## Universal To

This way, all the emails generated by your application will be sent to a specific address, instead of the address actually specified when sending the message. This can be done via the to option in your config/mail.php configuration file.

```php
'to' => [
    'address' => 'example@example.com',
    'name' => 'Example'
],
```

## Mailtrap

Use a service like Mailtrap and the smtp driver to send your email messages to a "dummy" mailbox where you may view them in a true email client.