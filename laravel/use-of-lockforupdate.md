---
description: Locking the database between read and update of a database record
---

# Use of lockForUpdate()

Suppose we have an application where many users or API transactions are creating invoices. Each of these invoices must have a sequential number and not be impacted by multiple updates happening on the database at the same time.

For the sake of this example, we have decided not to use the auto incrementing primary key of the database table.  Perhaps our application is multi-tenant and each tenant or project in our system has their own invoice sequence.

The naive approach is to simply read the current invoice number, increment it and save it back to the database.  The problem with this approach is that as we are reading the current invoice number, another user's request could also be reading the same value.  Then we both increment the value and write back to the database.  We now have two invoices in our system with the same value.

The better approach is to use a transaction closure and Eloquent's `lockForUpdate()`

In the case of MySQL, this adds `SELECT ... FOR UPDATE` to the original query meaning that the record is locked until the transaction is completed.  The simplest way to complete the transaction is to use the closure approach;

```php
$invoice = DB::transaction(function () use ($tenant) {
    $inv = DB::table('tenants')
        ->where('id', $tenant)
        ->lockForUpdate()
        ->first('next_invoice')
		    ->next_invoice;

    DB::table('tenants')
        ->where('id', $tenant)
        ->update(['next_invoice' => ++$inv]);

    return $inv;
});
```

This wraps the select and update in a transaction and applies the lockForUpdate to the select. It then writes back the incremented value to the database and returns the number just taken to the caller for use in the invoice generation.

