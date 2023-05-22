---
description: When one email should be sent using routeNotificationForMail
---

# Send Notification to all team members

### Scenario

A team receives a single notification, for instance as a single slack message, but the email version needs to be sent to all members of the team.

Looping over the notification for each member would send individual emails, but would also send multiple slack messages.

### Solution

Add the `use Notifiable` trait to the Team model

Add a method `routeNotificationForMail` to the Team model

```php
    public function routeNotificationForMail($notification)
    {
        return $this->users->pluck('name','email');
    }
```

This function returns a collection of recipients with the user email as the key and the user name as the value.

The Notification, sent by email will be \[To] the complete list of team members rather than as individual emails.&#x20;

An advantage of this approach is that all team members can see that the email was sent to them all.

The method routeNotificationForEmail is covered in the docs, but it is not clear how this should be used to return multiple recipients [https://laravel.com/docs/8.x/notifications#customizing-the-recipient](https://laravel.com/docs/8.x/notifications#customizing-the-recipient).

### Use with Jetstream Teams function

When using [Jetstream Teams](https://jetstream.laravel.com/2.x/features/teams.html) the team owner is not returned by the `$team->users()` relationship.  To obtain the full list including the owner, use the `allUsers()` function.

```php
    public function routeNotificationForMail($notification)
    {
        return $this->allUsers()->pluck('name','email');
    }
```

{% hint style="info" %}
**You can discuss pages on this site at** [**https://github.com/snapey/talltips/discussions**](https://github.com/snapey/talltips/discussions)
{% endhint %}
