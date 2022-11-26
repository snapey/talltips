---
description: When user enters Pounds & Pence or Dollars & Cents
---

# Using a mutator to save currency

When working with currencies, it's good practice to store the value in the lowest denomination (eg cents) and then revert to normal currency format when displaying the value.  This is to avoid the use of floating point numbers and rounding issues when calculating VAT percentages.

If you capture the value from the user, then they will present you will something like 12.75 when you need to save 1275 in the database.

Eloquent accessors and mutators can be used for this conversion in each direction.  The example below uses Pounds, but the principle is the same for Dollars or Euro.

In the eloquent model

```php
    public function setPoundsAttribute($value)
    {
        $this->each = strval($value*100);
    }

    public function getPoundsAttribute()
    {
        return number_format($this->each/100,2);
    }
```

`each` is the column storing the pence value.

Now we can save the value to the model in pounds format eg `$item->pounds = '12.75'`

{% hint style="info" %}
Be sure to add the mutated field (pounds) to your `$fillable` attribute if you are using mass assignment protection.
{% endhint %}

{% hint style="warning" %}
This question when posed on Laracasts produced quite a few answers and alternative ways, the main issue being to avoid floating point conversion issues with certain values, eg 19.99

[https://laracasts.com/discuss/channels/general-discussion/int-conversion-issue](https://laracasts.com/discuss/channels/general-discussion/int-conversion-issue)
{% endhint %}
