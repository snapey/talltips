---
description: When all you have is CRON
---

# Using Queues on Shared Hosting with Laravel

Sometimes, shared hosting must be used that does not permit the installation of supervisor to run the queue worker.

A common example of this are servers deployed with Cpanel access only.

One approach for non-time sensitive queue work (such as sending emails) is to add a task to the scheduler that starts the queue worker every minute;

```php
$schedule->command('queue:work --stop-when-empty')
             ->everyMinute()
             ->withoutOverlapping();
```

Adding this line to the scheduler in app\Console\Kernel.php and then setting up a cron job to run the scheduler, ensures that the queue gets serviced every minute.

For Cpanel servers, the Cron statement might look like;

```
* * * * * /usr/local/bin/php /home/{account_name}/live/artisan schedule:run
```

where `{account_name}` is the user account that cpanel is running under and `live` is the folder of the laravel application
