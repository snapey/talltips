---
description: When you need additional scope on notifications
---

# Adding column to Database Notifications table

Suppose your user can belong to many teams, and they have the ability to switch teams, you, like me, may want to allow the user to have unread notifications only for the team that they are currently in.

A similar situation may exist for multi-tenant applications where the same user can be a member of different tenants.

You may find that you need an additional column on the notifications table that you can then later filter with a global scope.

This article: [https://www.ystash.com/blog/extra-columns-with-laravel-database-notifications/](https://www.ystash.com/blog/extra-columns-with-laravel-database-notifications/) suggests creating a new database notification channel.

The approach presented here is simpler as it only involves listening to the Notification Eloquent model **creating** event using an [observer](https://laravel.com/docs/8.x/eloquent#observers).

### Add your additional column to the notifications table

This is fairly straightforward, just a regular migration targeting the notifications table

```php
    public function up()
    {
        Schema::table('notifications', function (Blueprint $table) {
            $table->foreignId('organisation_id');
        });
    }
```

### Create an observer

If you don't already have it, create app\Observers table

Create new file NotificationObserver.php

{% code title="app\Observers\NotificationObserver.php" %}
```php
<?php

namespace App\Observers;

class NotificationObserver
{
    public function creating($model)
    {
        // here set the column data for your new column
    }

}

```
{% endcode %}

### Add Observer to AppServiceProvider

In the boot() method of Providers/AppServiceProvider

```php
    public function boot()
    {
        DatabaseNotification::observe(NotificationObserver::class);
    }
```

Thats it! when your database notification is created, your additional data will be added
