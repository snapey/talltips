---
description: Not as simple as it may sound.  How to avoid problems from race conditions.
---

# Create Guaranteed Unique Invoice Number in Laravel

An interesting question was posed on Laracasts Discussions which seems straightforward, but comes with a number of hidden issues;

> I need help in generating invoice number
>
> I need this format. For example in the financial year 01-04-2022 to 31-03-2023 I have generate number like this R2022/2023-0001 and so on.
>
> In the next year 01-04-2023 to 31-03-2024 it should be like this R2023/2024-0001

## Problems Considered

* How do we determine the current financial year?
* How do we reset the count in each financial year?
* How do we increment the invoice number and make it atomic so that another user cannot get the same number

## Determine the current financial year

This is relatively straightforward.  Get today, then change to 1st April in the current calendar year.  If the date moved forward in time then today must be between 1st January and 31st March.  If so revert to the previous year.

```php
$dt = today()->setMonth(Carbon::APRIL)->setDay(1);

if($dt > today()) {
 $dt->subYear(1);
}
```

## Reset the count in each Financial Year

This is a little tricky as the invoice number must restart at 0001 in each year.

We don't want to have something that needs to run on a specific date when we can let our database do it for us.  We are going to create a database table to keep track of the current invoice number in each financial year;

### Model and Migration

```bash
php artisan make:model InvoiceSequence -m
```

In our invoice\_sequences migration

```php
        Schema::create('invoice_sequences', function (Blueprint $table) {
            $table->id();
            $table->integer('fy')->unique();
            $table->integer('current')->default(0);
        });
    
```

the `fy` column allows us to create a new record for each financial year.  The `current` column stores the next invoice number to be issued.  Unique on the `fy` column ensures that there can only be one entry per year.

### Eloquent FirstOrCreate

Initially, I was using the FirstOrCreate method to create a new record each time the financial year changes;

```php
$is = InvoiceSequence::firstOrCreate(['fy' => $dt->format('Y')]);
```

The `fy` column is set to the current financial year eg 2022.  When the year changes to 2023, there will not be a row with that year so a new row is added, and thanks to the default value, the `current` value is automatically set to 0.  However, it has been pointed out to me by @swaz at Laracasts that the firstOrCreate is not atomic and if two transactions occur at the same time then two entries might be created.  For this reason, \`unique()\` is added to the table.

see: [https://freek.dev/1087-breaking-laravels-firstorcreate-using-race-conditions](https://freek.dev/1087-breaking-laravels-firstorcreate-using-race-conditions)

The preferred approach is instead as below;

```php
$year = $dt->format('Y');

DB::statement("INSERT INTO invoice_sequences (fy) VALUES ({$year}) ON DUPLICATE KEY UPDATE year = year, id=LAST_INSERT_ID(id)");
    
$lastInsertId = DB::getPDO()->lastInsertId();
```

This attempts to insert a new row into the database, and if this fails, returns the record ID for the existing row.  The point is that this single SQL statement is atomic. Two threads cannot interfere with each other's operation.

## Safely Increment the invoice number

If we don't take care at this stage, two requests occurring at the same time could be given the same invoice number.

At first glance, it would be easy to write a solution that;

* gets the invoice\_sequences record for the current year
* adds one the current value
* saves the record

The problem with this approach is that two users might both execute the read part and get the same number, both then increment and both save. The counter has only been incremented once and both requests get the same number.

This is not a problem in testing, or when your application is small, but as it increases in activity the chances of this happening start to increase, are unpredictable and extremely hard to track down.

The example here is for invoice numbers but it could apply to any number that needs to be guaranteed to issue once and once only.

&#x20;The solution, inspired by this [https://www.sqlines.com/mysql/how-to/select-update-single-statement-race-condition](https://www.sqlines.com/mysql/how-to/select-update-single-statement-race-condition) safely increments the `current` number and also returns its value.

```php
DB::statement("UPDATE invoice_sequences SET current = LAST_INSERT_ID(current) + 1 WHERE id = {$lastInsertId}");

$current = DB::getPDO()->lastInsertId();
```

Read the linked article to understand why this works.

{% hint style="warning" %}
This solution is only applicable to MYSQL
{% endhint %}

## Putting it all together

Along with formatting it in the format requested by the question poster

```php
$dt = today()->setMonth(Carbon::APRIL)->setDay(1);

if($dt > today()) {
 $dt->subYear(1);
}

$year = $dt->format('Y');

// ensure there is a record for the current financial year
DB::statement("INSERT INTO invoice_sequences (fy) VALUES ({$year}) ON DUPLICATE KEY UPDATE year = year, id=LAST_INSERT_ID(id)");
$lastInsertId = DB::getPDO()->lastInsertId();

// automatically increment the count AND get the value
DB::statement("UPDATE invoice_sequences SET current = LAST_INSERT_ID(current) + 1 WHERE id = {$lastInsertId}");
$current = DB::getPDO()->lastInsertId();

$invoiceNumber = sprintf('R%s/%s-%04u', $year, $year+1, $current);
```

Grateful to @swaz at Laracasts for helping with this.  You can read the full thread and how we arrived at the solution here;

{% embed url="https://laracasts.com/discuss/channels/laravel/generating-unique-invoice-number-based-on-financial-yesr" %}

{% hint style="info" %}
**You can discuss pages on this site at** [**https://github.com/snapey/talltips/discussions**](https://github.com/snapey/talltips/discussions)****
{% endhint %}
