# Drivers / Configurations

The queue configuration file is stored in config/queue.php.

**Distinction between "connections" and "queues".**
* connections defines a particular connection to a backend service such as Amazon SQS, Beanstalk, or Redis.
* any given queue connection may have multiple "queues" which may be thought of as different stacks or piles of queued jobs.
* each connection configuration example in the queue configuration file contains a  queue attribute.
* This is the default queue that jobs will be dispatched to when they are sent to a given connection.
* if you dispatch a job without explicitly defining which queue it should be dispatched to, the job will be placed on the queue that is defined in the queue attribute of the connection configuration.

```php
// This job is sent to the default queue...
Job::dispatch();

// This job is sent to the "emails" queue...
Job::dispatch()->onQueue('emails');
```

## Drivers

### Database

```
php artisan queue:table

php artisan migrate
```

### Redis

In order to use the redis queue driver, you should configure a Redis database connection in `config/database.php` configuration file.

### Other Driver Prerequisites

Amazon SQS: aws/aws-sdk-php ~3.0
Beanstalkd: pda/pheanstalk ~3.0
Redis: predis/predis ~1.0

# Creating Jobs

## Generating Job Classes

All of the queueable jobs are stored in the `app/Jobs` directory.

`php artisan make:job ProcessPodcast`

The generated class will implement the `Illuminate\Contracts\Queue\ShouldQueue` interface, indicating to Laravel that the job should be pushed onto the queue to run asynchronously.

## Class Structure

```php
<?php

namespace App\Jobs;

use App\Podcast;
use App\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * Create a new job instance.
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }
}
```

# Dispatching Jobs

```php
class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast);
    }
}
```

## Delayed Dispatching

```php
class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)
                ->delay(now()->addMinutes(10));
    }
}
```

## Job Chaining

Job chaining allows you to specify a list of queued jobs that should be run in sequence. If one job in the sequence fails, the rest of the jobs will not be run.

```php
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch();
```

## Customizing The Queue & Connection

### Dispatching To A Particular Queue

By pushing jobs to different queues, you may "categorize" your queued jobs and even prioritize how many workers you assign to various queues.

```php
class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');
    }
}
```

### Dispatching To A Particular Connection

```php
class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');
    }
}
```

```php
ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```

## Specifying Max Job Attempts / Timeout Values

### Max Attempts

* Via artisan command

`php artisan queue:work --tries=3`

* Via job class itself

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 5;
}
```

### Time Based Attempts

As an alternative to defining how many times a job may be attempted before it fails, you may define a time at which the job should timeout. This allows a job to be attempted any number of times within a given time frame.

Add this to job class:

```php
/**
 * Determine the time at which the job should timeout.
 *
 * @return \DateTime
 */
public function retryUntil()
{
    return now()->addSeconds(5);
}

```

### Timeout

* the maximum number of seconds that jobs can run may be specified using the  --timeout switch on the Artisan command line

`php artisan queue:work --timeout=30`

OR

* in job class

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of seconds the job can run before timing out.
     *
     * @var int
     */
    public $timeout = 120;
}
```

## Rate Limiting

**This feature requires that your application can interact with a Redis server.**

This feature can be of assistance when your queued jobs are interacting with APIs that are also rate limited. For example, using the `throttle` method, you may throttle a given type of job to only run 10 times every 60 seconds. If a lock can not be obtained, you should typically release the job back onto the queue so it can be retried later:

```php
Redis::throttle('key')->allow(10)->every(60)->then(function () {
    // Job logic...
}, function () {
    // Could not obtain lock...

    return $this->release(10);
});
```

Alternatively, you may specify the maximum number of workers that may simultaneously process a given job. This can be helpful when a queued job is modifying a resource that should only be modified by one job at a time. For example, using the funnel method, you may limit jobs of a given type to only be processed by one worker at a time:

```php
Redis::funnel('key')->limit(1)->then(function () {
    // Job logic...
}, function () {
    // Could not obtain lock...

    return $this->release(10);
});
```

## Error Handling

If an exception is thrown while the job is being processed, the job will automatically be released back onto the queue so it may be attempted again.

# Running The Queue Worker

`php artisan queue:work`

### Processing A Single Job

`php artisan queue:work --once`

### Specifying The Connection & Queue

`php artisan queue:work redis`

`php artisan queue:work redis --queue=emails`

## Resource considerations

**Daemon queue workers do not "reboot" the framework before processing each job. Therefore, you should free any heavy resources after each job completes. For example, if you are doing image manipulation with the `GD` library, you should free the memory with `imagedestroy` when you are done.**

## Queue Priorities

To start a worker that verifies that all of the high queue jobs are processed before continuing to any jobs on the low queue, pass a comma-delimited list of queue names to the work command:

`php artisan queue:work --queue=high,low`

## Queue Workers & Deployment

`php artisan queue:restart`

## Job Expirations & Timeouts

### Job Expiration

* In config/queue.php each queue connection defines a retry_after option.
* This option specifies how many seconds the queue connection should wait before retrying a job that is being processed.
* Typically, you should set the retry_after value to the maximum number of seconds your jobs should reasonably take to complete processing.

### Worker Timeouts

* The --timeout option specifies how long the Laravel queue master process will wait before killing off a child queue worker that is processing a job.
* Sometimes a child queue process can become "frozen" for various reasons, such as an external HTTP call that is not responding.

`php artisan queue:work --timeout=60`

### Worker Sleep Duration

* When jobs are available on the queue, the worker will keep processing jobs with no delay in between them.
* However, the sleep option determines how long the worker will "sleep" if there are no new jobs available. While sleeping, the worker will not process any new jobs - the jobs will be processed after the worker wakes up again.
  
`php artisan queue:work --sleep=3`

# Supervisor

## Installing Supervisor

`sudo apt-get install supervisor`

## Configuring Supervisor

* config file is stored in `/etc/supervisor/conf.d` directory.
* `laravel-worker.conf` sample:

```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```

## Starting Supervisor

```
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start laravel-worker:*
```

# Dealing With Failed Jobs

Create table to save failed jobs:

```
php artisan queue:failed-table

php artisan migrate
```

## Cleaning Up After Failed Jobs

Define a failed method directly on your job class, allowing you to perform job specific clean-up when a failure occurs.
This is the perfect location to send an alert to your users or revert any actions performed by the job. 

```php
<?php

namespace App\Jobs;

use Exception;
use App\Podcast;
use App\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * Create a new job instance.
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }

    /**
     * The job failed to process.
     *
     * @param  Exception  $exception
     * @return void
     */
    public function failed(Exception $exception)
    {
        // Send user notification of failure, etc...
    }
}
```

## Failed Job Events

* register an event that will be called when a job fails.
* use Queue::failing method.
* This event is a great opportunity to notify your team via email or HipChat.

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Queue\Events\JobFailed;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Queue::failing(function (JobFailed $event) {
            // $event->connectionName
            // $event->job
            // $event->exception
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

## Retrying Failed Jobs

view all of your failed jobs that have been inserted into your failed_jobs database table.

`php artisan queue:failed`

The queue:failed command will list the job ID, connection, queue, and failure time. The job ID may be used to retry the failed job.

`php artisan queue:retry 5`

Or all

`php artisan queue:retry all`

Delete a failed job.

`php artisan queue:forget 5`

Delete all:

`php artisan queue:flush`

# Job Events

* Use the `before` and `after` methods on the Queue facade
* specify callbacks to be executed before or after a queued job is processed.
* These callbacks are opportunity to perform additional logging or increment statistics for a dashboard.

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobProcessing;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Queue::before(function (JobProcessing $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });

        Queue::after(function (JobProcessed $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
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

