---
description: Domain specific settings
---

# Using Spatie Valuestore to hold frequently accessed settings

This was a reply posted to the [Laracasts forum](https://laracasts.com/discuss/channels/tips/code-review-idea-to-solve-problem).  I know I will use this in the future, so documented here also as a reminder to myself as much as anything.

Install Valuestore `composer require spatie/valuestore`

In AppServiceProvider register method;

```php
 public function register()
 {
   $this->app->singleton('valuestore', function () {
     return \Spatie\Valuestore\Valuestore::make(storage_path('app/settings.json'));
   });

   $values = $this->app->valuestore->all();

   $this->app->bind('settings', function () use($values) {
     return $values;
   });
 }
```

What we have in the app container is a singleton that points to the `valuestore` class.  When you use that, you are directly interacting with the settings stored in the file.

When you use the `settings` bound to the app container, you are using a cached version of the values as they were at the start of the request cycle (as an associated array).

So instead of writing the current Euro rate to a database row, put it in the valuestore instead;

```php
app('valuestore')->put('EUR', $rate)
```

and in your model when you want to apply this;

```php
public function getPriceEUR()
{
  return intval($this->usd_price / app('settings')['EUR'] * 100);}
}
```

By using `settings` and not `valuestore` the file will only be accessed once and not for each iterated product in a collection.

Of course you now have a place where you can store other currencies or any other application settings using the full features of the [Valuestore](https://github.com/spatie/valuestore) package.\


{% hint style="info" %}
Be careful with decimals when converting . See [https://laracasts.com/discuss/channels/general-discussion/int-conversion-issue](https://laracasts.com/discuss/channels/general-discussion/int-conversion-issue)
{% endhint %}
