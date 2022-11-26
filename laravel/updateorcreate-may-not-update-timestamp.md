---
description: When the record is identical, the updated_at timestamp is not changed
---

# UpdateOrCreate may not update timestamp

Consider the following code which is part of a spreadsheet import of a product catalogue.

```php
    Item::updateOrCreate(
        ['code' => $row[0]],
        [
            'description'=> $row[1],
            'category'=> $row[2],
            'uom' => $row[3],
            'case_quantity' => $row[4],
            'each' => intval(strval($row[5]*100)),
            'generic' => false
    ]);
```

Using the Eloquent `model:updateOrCreate()`, if the spreadsheet contains any new items then a new record is added to the product catalogue.

If the Item already exists, but for instance, the price has changed then the existing record is found and updated.

However if the row in the spreadsheet is IDENTICAL to the record already in the database, then Eloquent knows that the record does not have any changes (is not dirty) and skips the write process. In this scenario, the `updated_at` field retains the previous value.

This might be ok for your use case, but in one project, the updated\_at column was being used as part of a scope to show only _current_ products to the user.

The solution is to add `updated_at` to the data to be written to force a new value, and to ensure that updated\_at is in the `$fillable` array or unguarded.

```php
    Item::updateOrCreate(
        ['code' => $row[0]],
        [
            'description'=> $row[1],
            'category'=> $row[2],
            'uom' => $row[3],
            'case_quantity' => $row[4],
            'each' => intval(strval($row[5]*100)),
            'generic' => false,
            'updated_at' => now(),
    ]);
```
