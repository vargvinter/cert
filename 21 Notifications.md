# Creating Notifications

`php artisan make:notification InvoicePaid`

* This command will place a fresh notification class in your `app/Notifications` directory.

# Sending Notifications

## Using The Notifiable Trait

```php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;
}
```

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

## Using The Notification Facade

This is useful primarily when you need to send a notification to multiple notifiable entities such as a collection of users. 

```php
Notification::send($users, new InvoicePaid($invoice));
```

## Specifying Delivery Channels

```php
/**
 * Get the notification's delivery channels.
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function via($notifiable)
{
    return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
}
```

## Queueing Notifications

 speed up your application's response time, let your notification be queued by adding the ShouldQueue interface and Queueable trait to your class.
 
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;

class InvoicePaid extends Notification implements ShouldQueue
{
 use Queueable;

 // ...
}
```

If you would like to delay the delivery of the notification, you may chain the delay method onto your notification instantiation:

```php
$when = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($when));
```

## On-Demand Notifications

Sometimes you may need to send a notification to someone who is not stored as a "user" of your application.

```php
Notification::route('mail', 'taylor@laravel.com')
            ->route('nexmo', '5555555555')
            ->notify(new InvoicePaid($invoice));
```

# Mail Notifications

## Formatting Mail Messages

* Define a `toMail` method on the notification class.
* This method will receive a $notifiable entity and should return a  `Illuminate\Notifications\Messages\MailMessage` instance. 

```php
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->greeting('Hello!')
                ->line('One of your invoices has been paid!')
                ->action('View Invoice', $url)
                ->line('Thank you for using our application!');
}
```

In this example, we register a greeting, a line of text, a call to action, and then another line of text. These methods provided by the MailMessage object make it simple and fast to format small transactional emails. The mail channel will then translate the message components into a nice, responsive HTML email template with a plain-text counterpart. 

### Other Notification Formatting Options

```php
public function toMail($notifiable)
{
    return (new MailMessage)->view(
        'emails.name', ['invoice' => $this->invoice]
    );
}
```

In addition, you may return a mailable object from the toMail method:

```php
public function toMail($notifiable)
{
    return (new Mailable($this->invoice))->to($this->user->email);
}
```

### Error Messages

You may indicate that a mail message is regarding an error by calling the error method when building your message.

```php
public function toMail($notifiable)
{
    return (new MailMessage)
                ->error()
                ->subject('Notification Subject')
                ->line('...');
}
```

## Customizing The Recipient

You may customize which email address is used to deliver the notification by defining a routeNotificationForMail method on the entity:

```php
class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the mail channel.
     *
     * @return string
     */
    public function routeNotificationForMail()
    {
        return $this->email_address;
    }
}
```

## Customizing The Subject

```php
public function toMail($notifiable)
{
    return (new MailMessage)
                ->subject('Notification Subject')
                ->line('...');
}
```

## Customizing The Templates

After running this command, the mail notification templates will be located in the resources/views/vendor/notifications directory

`php artisan vendor:publish --tag=laravel-notifications`

# Markdown

## Generating The Message

`php artisan make:notification InvoicePaid --markdown=mail.invoice.paid`

Like all other mail notifications, notifications that use Markdown templates should define a  toMail method on their notification class. However, instead of using the line and action methods to construct the notification, use the markdown method to specify the name of the Markdown template that should be used:

```php
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

## Writing The Message

Like in 20 Mail.md

# Database Notifications

## Prerequisites

* stores the notification information in a database table.
* will contain information such as the notification type as well as custom JSON data.

```
php artisan notifications:table

php artisan migrate
```

## Formatting Database Notifications

define a toDatabase or  toArray method on the notification class.

```php
public function toArray($notifiable)
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

`toDatabase` Vs. `toArray`

The toArray method is also used by the broadcast channel to determine which data to broadcast to your JavaScript client. If you would like to have two different array representations for the database and broadcast channels, you should define a toDatabase method instead of a toArray method.

## Accessing The Notifications

The Illuminate\Notifications\Notifiable trait, includes a notifications Eloquent relationship that returns the notifications for the entity.

* By default, notifications will be sorted by the created_at timestamp:

```php
$user = App\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

If you want to retrieve only the "unread" notifications, you may use the unreadNotifications relationship.

```php
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

## Marking Notifications As Read

The  Illuminate\Notifications\Notifiable trait provides a markAsRead method, which updates the  read_at column on the notification's database record:

```php
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

OR

```php
$user->unreadNotifications->markAsRead();
```

OR 

```php
$user = App\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

Of course, you may delete the notifications to remove them from the table entirely:

```php
$user->notifications()->delete();
```

# Broadcast Notifications

## Formatting Broadcast Notifications

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the broadcastable representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return BroadcastMessage
 */
public function toBroadcast($notifiable)
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

### Broadcast Queue Configuration

```php
return (new BroadcastMessage($data))
                ->onConnection('sqs')
                ->onQueue('broadcasts');
```

## Listening For Notifications

Notifications will broadcast on a private channel formatted using a {notifiable}.{id} convention. So, if you are sending a notification to a App\User instance with an ID of 1, the notification will be broadcast on the App.User.1 private channel. When using Laravel Echo, you may easily listen for notifications on a channel using the notification helper method:

```js
Echo.private('App.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

### Customizing The Notification Channel

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The channels the user receives notification broadcasts on.
     *
     * @return string
     */
    public function receivesBroadcastNotificationsOn()
    {
        return 'users.'.$this->id;
    }
}
```

# SMS Notifications

## Prerequisites

* install the `nexmo/client` Composer package.
* add a few configuration options to your `config/services.php` configuration file.

```php
'nexmo' => [
    'key' => env('NEXMO_KEY'),
    'secret' => env('NEXMO_SECRET'),
    'sms_from' => '15556666666',
],
```

## Formatting SMS Notifications

```php
/**
 * Get the Nexmo / SMS representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your SMS message content');
}
```

### Unicode Content

If your SMS message will contain unicode characters, you should call the unicode method when constructing the NexmoMessage instance:

```php
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your unicode message')
                ->unicode();
}
```

## Customizing The "From" Number

```php
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your SMS message content')
                ->from('15554443333');
}
```

## Routing SMS Notifications

When sending notifications via the nexmo channel, the notification system will automatically look for a phone_number attribute on the notifiable entity.
Customize it:

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Nexmo channel.
     *
     * @return string
     */
    public function routeNotificationForNexmo()
    {
        return $this->phone;
    }
}
```

# Slack Notifications

## Prerequisites

`composer require guzzlehttp/guzzle`

## Formatting Slack Notifications

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->content('One of your invoices has been paid!');
}
```

### Customizing The Sender & Recipient

You may use the from and to methods to customize the sender and recipient. The from method accepts a username and emoji identifier, while the to method accepts a channel or username:

```php
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->from('Ghost', ':ghost:')
                ->to('#other')
                ->content('This will be sent to #other');
}
```

You may also use an image as your logo instead of an emoji:

```php
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->from('Laravel')
                ->image('https://laravel.com/favicon.png')
                ->content('This will display the Laravel logo next to the message');
}
```

## Slack Attachments

```php
public function toSlack($notifiable)
{
    $url = url('/exceptions/'.$this->exception->id);

    return (new SlackMessage)
                ->error()
                ->content('Whoops! Something went wrong.')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Exception: File Not Found', $url)
                               ->content('File [background.jpg] was not found.');
                });
}
```

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/invoices/'.$this->invoice->id);

    return (new SlackMessage)
                ->success()
                ->content('One of your invoices has been paid!')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Invoice 1322', $url)
                               ->fields([
                                    'Title' => 'Server Expenses',
                                    'Amount' => '$1,234',
                                    'Via' => 'American Express',
                                    'Was Overdue' => ':-1:',
                                ]);
                });
}
```

### Markdown Attachment Content

```php
public function toSlack($notifiable)
{
    $url = url('/exceptions/'.$this->exception->id);

    return (new SlackMessage)
                ->error()
                ->content('Whoops! Something went wrong.')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Exception: File Not Found', $url)
                               ->content('File [background.jpg] was *not found*.')
                               ->markdown(['text']);
                });
}
```

## Routing Slack Notifications

```php
class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     *
     * @return string
     */
    public function routeNotificationForSlack()
    {
        return $this->slack_webhook_url;
    }
}
```

# Custom Channels

* define a class that contains a `send` method.
* The method should receive two arguments: a  `$notifiable` and a `$notification`

```php
<?php

namespace App\Channels;

use Illuminate\Notifications\Notification;

class VoiceChannel
{
    /**
     * Send the given notification.
     *
     * @param  mixed  $notifiable
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return void
     */
    public function send($notifiable, Notification $notification)
    {
        $message = $notification->toVoice($notifiable);

        // Send notification to the $notifiable instance...
    }
}
```

* Once your notification channel class has been defined, you may return the class name from the  via method of any of your notifications

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use App\Channels\VoiceChannel;
use App\Channels\Messages\VoiceMessage;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;

class InvoicePaid extends Notification
{
    use Queueable;

    /**
     * Get the notification channels.
     *
     * @param  mixed  $notifiable
     * @return array|string
     */
    public function via($notifiable)
    {
        return [VoiceChannel::class];
    }

    /**
     * Get the voice representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return VoiceMessage
     */
    public function toVoice($notifiable)
    {
        // ...
    }
}
```

