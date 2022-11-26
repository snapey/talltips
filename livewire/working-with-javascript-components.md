---
description: Updating Livewire using Bootstrap DatePicker
---

# Working with Javascript Components

{% hint style="warning" %}
Not TALL stack, but this note useful for Livewire + Bootstrap projects when trying to bring Livewire to an older project&#x20;
{% endhint %}

{% hint style="success" %}
If possible, just use the native controls such as `type='date'`
{% endhint %}

When using Bootstrap date picker, just using wire:model on the field is not going to work. Livewire needs telling that the Input field has changed.

```markup
<input 
    wire:model="taskduedate"
    type="text" class="form-control datepicker" placeholder="Due Date" autocomplete="off"
    data-provide="datepicker" data-date-autoclose="true" data-date-format="mm/dd/yyyy" data-date-today-highlight="true"                        
    onchange="this.dispatchEvent(new InputEvent('input'))"
>
```

Livewire is looking for an input event to know that the field is dirty. The date picker seems to not trigger any input events, but it does trigger a regular input onchange event. We can hook into this and dispatch an input event. This causes Livewire to sync the field with the server.

This strategy may work with other similar components.
