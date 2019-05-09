# Defining Broadcast Events

* Implement the  `Illuminate\Contracts\Broadcasting\ShouldBroadcast` interface on the event class.
* The `ShouldBroadcast` interface requires you to implement a single method: `broadcastOn`.
* The  `broadcastOn` method should return a channel or array of channels that the event should broadcast on.
* Channels instances: `Channel` (public), `Private`, `Presence` (both private).

```php
class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    public $user;

    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('user.'.$this->user->id);
    }
}
```

## Broadcast name

Define `broadcastAs` method on the event.

```php
public function broadcastAs()
{
    return 'server.created';
}
```

Register listener with a leading `.` character. This will instruct Echo to not prepend the application's namespace to the event:

```javascript
.listen('.server.created', function (e) {
    //....
});
```

## Broadcast Data

* `public` properties are automatically serialized and broadcast as the event's payload.
* To add more data add `broadcastWith` to event class.

```php
public function broadcastWith()
{
    return ['id' => $this->user->id];
}
```

## Broadcast Queue

* By default, each broadcast event is placed on the default queue for the default queue connection specified in `queue.php`.
* To customize use `broadcastQueue`:

```php
public $broadcastQueue = 'your-queue-name';
```

To broadcast event using the `sync` queue instead of the default queue driver, implement the `ShouldBroadcastNow` interface instead of `ShouldBroadcast`:

```php
<?php

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class ShippingStatusUpdated implements ShouldBroadcastNow
{
    //
}
```

## Broadcast Conditions

* broadcast event only if a given condition is true.
* define these conditions by adding a `broadcastWhen` method

```php
public function broadcastWhen()
{
    return $this->value > 100;
}
```

# Authorizing Channels

* Private channels require to authorize that the currently authenticated user can actually listen on the channel.
* This is accomplished by making an HTTP request to Laravel application with the channel name and allowing application to determine if the user can listen on that channel.

## Defining Authorization Routes

* `BroadcastServiceProvider` included with Laravel application, make a call to the `Broadcast::routes` method. 
* This method will register the `/broadcasting/auth` route to handle authorization requests.

```php
Broadcast::routes();
```

* pass an array of route attributes to the method if you would like to customize the assigned attributes.

```php
Broadcast::routes($attributes);
```

## Defining Authorization Callbacks

* define the logic that will actually perform the channel authorization in `routes/channels.php`

```php
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

The `channel` method accepts two arguments: the name of the channel and a callback which returns `true` or `false` indicating whether the user is authorized to listen on the channel.

All authorization callbacks receive the currently authenticated user as their first argument and any additional wildcard parameters as their subsequent arguments. In this example, we are using the `{orderId}` placeholder to indicate that the "ID" portion of the channel name is a wildcard.

## Authorization Callback Model Binding

```php
use App\Order;

Broadcast::channel('order.{order}', function ($user, Order $order) {
    return $user->id === $order->user_id;
});
```

# Broadcasting Events

```php
event(new ShippingStatusUpdated($update));
```

## Only To Others

* Replace the `event` function with the `broadcast` function.
* Like the `event` function, the `broadcast` function dispatches the event to server-side listeners.

```php
broadcast(new ShippingStatusUpdated($update));
```

* `broadcast` function also exposes the `toOthers` method which allows to exclude the current user from the broadcast's recipients.

```php
broadcast(new ShippingStatusUpdated($update))->toOthers();
```

Example:
* Task list app. 
* To create a task, your application might make a request to a /task end-point which broadcasts the task's creation and returns a JSON representation of the new task. 
* Directly insert the new task into its task list.
* we also broadcast the task's creation.
* If  JavaScript application is listening for this event in order to add tasks to the task list, you will have duplicate tasks in your list: one from the end-point and one from the broadcast.
 
 ### Configuration
 
 * When utilizing a Laravel Echo instance, a socket ID is assigned to the connection.
 * When using `Vue` and `Axios` socket ID is attached to every outgoing request as `X-Socket-ID` header.
 * When `toOthers` method is called Laravel will extract socket id from the header and instruct the broadcaster to not broadcast to any connections with that socket ID.
                                                                                   
* When not using Axios and Vue manually configure JS app to send `X-Socket-ID`.
* retrieve the socket ID:

```javascript
var socketId = Echo.socketId();
```

# Receiving Broadcasts

* `npm install --save laravel-echo pusher-js`
* in `resources/assets/js/bootstrap.js`

```javascript
import Echo from "laravel-echo"

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key'
});
```

## Listening For Events

* Public channel:

```javascript
Echo.channel('orders')
    .listen('OrderShipped', (e) => {
        console.log(e.order.name);
    });
```

* Private channel with chain

```javascript
Echo.private('orders')
    .listen(...)
    .listen(...)
    .listen(...);
```

## Leaving A Channel

```javascript
Echo.leave('orders');
```

## Namespaces

* in the examples above that we did not specify the full namespace for the event classes.
* Echo will automatically assume the events are located in the  `App\Events` namespace.
* Configure it when instantiate Echo:

```javascript
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    namespace: 'App.Other.Namespace'
});
```

* Alternatively, prefix event classes with a `.` when subscribing to them using Echo.

```javascript
Echo.channel('orders')
    .listen('.Namespace.Event.Class', (e) => {
        //
    });
```

# Presence Channels

These are private channels while exposing the additional feature of awareness of who is subscribed to the channel.

## Authorizing Presence Channels

* When defining authorization callbacks for presence channels, don't return `true` if the user is authorized to join the channel. Instead, return an array of data about the user.
* The data returned by the authorization callback will be made available to the presence channel event listeners in JS app. If the user is not authorized to join the presence channel, return `false` or `null`:

```php
Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

## Joining Presence Channels

The `join` method will return a  `PresenceChannel` implementation which, along with exposing the `listen` method, allows to subscribe to the `here`, `joining`, and `leaving` events.

```javascript
Echo.join(`chat.${roomId}`)
    .here((users) => {
        //
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    });
```

* The `here` callback will be executed immediately once the channel is joined successfully.
* The `joining` method will be executed when a new user joins a channel
* The `leaving` method will be executed when a user leaves the channel.

## Broadcasting To Presence Channels

```php
public function broadcastOn()
{
    return new PresenceChannel('room.'.$this->message->room_id);
}
```

```php
broadcast(new NewMessage($message));

// To exclude current user:

broadcast(new NewMessage($message))->toOthers();
```

In JS app:

```javascript
Echo.join(`chat.${roomId}`)
    .here(...)
    .joining(...)
    .leaving(...)
    .listen('NewMessage', (e) => {
        //
    });
```

# Client Events

* broadcast an event to other connected clients without hitting Laravel application at all.
* useful for things like "typing" notifications.

```javascript
Echo.private('chat')
    .whisper('typing', {
        name: this.user.name
    });
```

* To listen for client events, use the `listenForWhisper` method:

```javascript
Echo.private('chat')
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

