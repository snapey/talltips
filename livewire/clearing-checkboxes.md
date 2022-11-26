---
description: When setting empty array does not clear checkboxes
---

# Clearing checkboxes in Livewire

I encountered an issue where checkboxes are not cleared when the array they are bound to is cleared.

Checkbox input elements like this;

```markup
<input wire:model="recipients.{{ $contact->id }}" 
    type="checkbox" 
    name="recipient-{{ $contact->id }}" 
    id="recipient-{{ $contact->id }}"
>
```

{% hint style="info" %}
the array of recipients uses dotted notation to specify the index
{% endhint %}

Recipients are contacts in the system and are output as an array for the user to select a number of recipients and then perform the action.  As each recipient is checked a key/value pair is added to the array.

```php
array:2 [
  42 => true
  34 => true
]
```

Checkboxes that have not been selected are not present in the array. Selecting then deselecting a checkbox causes its entry to be set false;

```php
array:2 [
  42 => false
  34 => true
]
```

Once the user has selected a number of recipients and then performed the activity, the checkboxes should be cleared.  Setting the `$recipients` to an empty array does nothing to the view. The previously checked checkboxes are still checked following render.

The solution to clear the checkboxes is to iterate through the array in the Livewire component, setting recipients to false;

```php
foreach($this->recipients as &$recipient) {
    $recipient = false;
}
```

_note the use of & to pass the array entry by reference._
