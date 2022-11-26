---
description: >-
  Through a simple addition to the user model we can enable application features
  for specific users only
---

# Super Simple User Based Feature Flags

When developing a new feature, it can be very useful to be able to get this feature in the hands of trusted users whilst all other users see nothing different.  This type of functionality can be termed a **Feature Switch** (or feature toggle). The method implemented here is easy to implement and robust in use.

There are only three components to this implementation

1. A config section that indicates if the feature is globally available
2. A function on the User model which tells us if the user has that feature available to them
3. Use of the function in views and classes to guide the workflow or enable view sections

Of course this approach may not suit all, and attention has to be paid to the effect of database changes for the feature which may not be available to all users.

### The config section

You could have a dedicated config file for feature switches, but in this case we use the app.php file and add a new section to it;

{% code title="config/app.php" %}
```php
    /*
    | Features
    | NU = Nudges are enabled
    */
    'features' => [
        'SE',  // Search is enabled
        'FL',  // Flashcards are enabled
    ],
      
```
{% endcode %}

Features that are globally available (available to all users) are added to the features section.  The comment tells us what other features are available but not yet implemented. &#x20;

You will start out with this features array empty, and instead give the flag to the test users. After you are happy with the new feature and want to roll it out to all users, add the appropriate flag to the features array.

### The User model

The addition of a simple function `hasFeature` will tell us if the feature is turned on for this user or turned on for everyone.

{% code title="User.php" %}
```php
    public function hasFeature($code)
    {
        // ignores user setting if the feature code is globally enabled.
        if(collect(config('app.features' ?? [] ))->contains($code)){
            return true;
        }

        return !! collect(explode(',',$this->features))->contains($code);
    }
```
{% endcode %}

If the global config contains the passed flag then the feature is available, irrespective of what the User model holds.  If the value is not in config, then check if the User model contains this value.

A simple `string` column is added to the users table containing `features` in this field place a comma separated  list of features available to this user, ie 'FL,NU'.

### Restricting the use of the feature

In views, a simple `@if` directive may be used. For example;

```php
@if(Auth::user()->hasFeature('SE'))
    <a href="{{ route('search') }}" class="px-2 py-2 text-sm font-bold text-center">SEARCH</a>
@endif
```

In classes and controllers, since the function simply returns true or false, we can use if, case statements, ternary statements etc

```php
if($user->hasFeature('NU')){
  // something to do with feature
}
```

### Cleaning up afterwards

Once you have proven your new feature and it has been rolled out globally for a period of time, it is best-practice to plan to remove the feature switch from your code now that it is no longer required.



