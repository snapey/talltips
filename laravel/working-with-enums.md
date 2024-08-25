---
description: Tips and handy utilities for working with enums in your Laravel project
---

# Working with Enums

## Making your Enum invokable

When working with enums, its handy to reference them statically such as when authorizing access to a controller method using a role name.

Personally, I find it clumsy to have to access the Enum static method such as&#x20;

&#x20;   `$this->authorize(Role::MANAGER->name());`

With a tiny function we can simplify this to

&#x20;   `$this->authorize(Role::MANAGER());`

This is because we can make the enum invokable using the magic `__callStatic()` method on the enum. This method is called when a static method is requested but does not exist. Using this we can return an instance of the Enum being used.

An example of an enum class for expenses, using this method;

{% code title="App\Enums\Expenses.php" %}
```php
<?php

namespace App\Enums;

use Illuminate\Support\Arr;

enum Expenses
{
  case CREATE_EXPENSE;
  case DELETE_EXPENSE;
  case APPROVE_EXPENSE;

  public static function __callStatic($name, $args)
  {
    $case = Arr::first(static::cases(), fn($case) => $case->name === $name);

    throw_unless($case, 'Missing Enum Case');

    return $case instanceof BackedEnum ? $case->value : $case->name;
  }

```
{% endcode %}

Now&#x20;

```php
Expenses::CREATE_EXPENSE()   // "CREATE_EXPENSE"
```

{% hint style="warning" %}
Why not just type CREATE\_EXPENSE directly as a string?  Well, the power of enums (and hopefully why you are using them) is because you cannot accidentally use a string which is not declared in your application  Expenses::CRAETE\_EXPENSE() will throw and exception whereas "CRAETE\_EXPENSE" will not.
{% endhint %}

{% hint style="success" %}
Adding this method to your enums does not remove any existing enum abilities
{% endhint %}

### Invokable Enum with Backed Enums

The method shown supports both plain Enums and Backed Enums

A Backed Enum returns a value (string or int) rather than the enum name

For instance, if we wanted to just work with ints in the database, our Enum might be declared as&#x20;

```php
enum Expenses: int
{
  case CREATE_EXPENSE = 1;
  case DELETE_EXPENSE = 2;
  case APPROVE_EXPENSE = 3;
```

Now;

```php
Expenses::CREATE_EXPENSE()   // 1
```

The method returns the name or the value, depending on whether your enum is backed.

