---
description: Where to place a custom mail theme in Laravel
---

# Specify a different mail theme for Notifications

{% hint style="info" %}
This issue was resolved in Laravel 8.5 [https://github.com/laravel/framework/issues/34391](https://github.com/laravel/framework/issues/34391)
{% endhint %}

Views can use something called _Hints_ to tell the framework where to look for the view file. The theme always uses a hint of `Mail::` which corresponds to the `resources/views/vendor/mail/html` folder. What if we want to locate our theme somewhere else?

We can define our own hint and bind it into the system in the `AppServiceProvider` with;

{% code title="app/Providers/AppServiceProvider.php" %}
```php
    public function boot()
    {
        $this->loadViewsFrom(resource_path('views/mail'), 'appmail');
    }

```
{% endcode %}

With this, we are saying that any view file that is hinted with `appmail::` is found in the `views/mail` folder.

Then, in the Notification, add the hint in the theme function;

```php
  public function toMail($notifiable)
  {
    return (new MailMessage)
      ->theme('appmail::mycorp')
      ->subject('Password Reset')
      ->line('Your password reset code is specified below')
      
    // etc
```

