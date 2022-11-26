---
description: Use SweetAlert2 to display animated popup alert following Livewire action
---

# SweetAlert2 with Livewire

Install SweetAlert in master layout

On views that expect to use SweetAlert, or in master layout file, add a listener

```javascript
window.addEventListener('swal',function(e){ 
    Swal.fire(e.detail);
});
```

Then trigger an event from Livewire component when you want to show the popup

```php
$this->dispatchBrowserEvent('swal', ['title' => 'Feedback Saved']);
```

Pass any SweetAlert config elements to style your popup

```php
$this->dispatchBrowserEvent('swal', [
	'title' => 'Feedback Saved',
	'timer'=>3000,
	'icon'=>'success',
	'toast'=>true,
	'position'=>'top-right'
]);
```

{% embed url="https://sweetalert2.github.io/#usage" %}

You might instead use this package;

{% embed url="https://github.com/jantinnerezo/livewire-alert" %}

