---
description: Populate a form with user's previous entry, or default data
---

# Using the old() helper

When we are presenting the user with a dropdown, from which they can choose, for example, a different user, you have to send your form all the possible choices. You can either send the whole model, or just `->pluck('name',$id')`

Then you loop over all the contacts setting the select value and name;

```php
<select name="contact">
@foreach($contacts as $contact)

	<option value="{{$contact->id}}">{{ $contact->name }}</option>

@endforeach
</select>
```

### Selecting the previous value

At each iteration of the loop, you want to check if the current contact (the one you are on in the loop) matches the id of the one in the model. We can do this by comparing the current ID with that already stored in the database.

```php
<select name="contact">
@foreach($contacts as $contact)
	<option value="{{$contact->id}}" 
			{{ $contact->id == $order->contact_id ? 'selected' : ''}}>
		  {{ $contact->name }}
   </option>

@endforeach
</select>
```

The ternary here compares the current contact with the contact on the order.

{% hint style="info" %}
This is the same as writing

```php
@if($contact->id == $order->contact_id) selected @endif
```
{% endhint %}

The above selects the current user on the order, but it does not remember the selection if you change it and then have a validation error.

This is where the old() helper comes in.

```php
<select name="contact">
@foreach($contacts as $contact)

	<option value="{{$contact->id}}" 
		{{ $contact->id == old('contact',$order->contact_id) ? 'selected' : ''}}>
		{{ $contact->name }}
        </option>

@endforeach
</select>
```

(assuming the select option is named 'contact')

Now, the current option is compared to whatever the old function returns. If it is not set (this is the first time the form is displayed, then the database value from the order is returned. If old data exists then this is being displayed as a result of a validation error, so it returns the value the person selected last time they posted the form.

{% hint style="info" %}
Syntax  `old( previous , default )`
{% endhint %}
