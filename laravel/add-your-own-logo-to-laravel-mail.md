---
description: Amending the template is easy to add your logo to the mail header
---

# Add your own logo to Laravel Mail

The mail template will already automatically add your application name to the mail template, but if you want to display your own logo?

### Publish the email stubs&#x20;

Run the command `php artisan vendor:publish --tag=laravel-mail`

This will copy the email layout stubs to the folder /resources/views/vendor/mail/html

### Edit the header.blade.php file.

To start with the template looks like;

```html
<tr>
<td class="header">
<a href="{{ $url }}" style="display: inline-block;">
@if (trim($slot) === 'Laravel')
<img src="https://laravel.com/img/notification-logo.png" class="logo" alt="Laravel Logo">
@else
{{ $slot }}
@endif
</a>
</td>
</tr>

```

This is the part that will show the laravel logo if the app name is laravel, but we can strip that out and add our own image.

```html
<tr>
<td class="header">
<a href="{{ $url }}" style="display: inline-block;">
<img src="{{ asset('/my-app-logo-192x192.png') }}" class="logo" alt="{{ $slot }}">
</a>
</td>
</tr>
```

The above links to your image.  Instead you can _embed_ the image

### Embedding your image instead of linking

Instead of linking to an image on your site, you may prefer to embed the image. See the following article regarding the pros and cons of different ways of adding images to your emails;

{% embed url="https://sendgrid.com/blog/embedding-images-emails-facts/" %}

Instead of linking to the image, you can base64 encode it;

```html
<tr>
<td class="header">
<a href="{{ $url }}" style="display: inline-block;">
<img src="data:image/png;base64,{{ base64_encode(file_get_contents(public_path('my-app-logo-192x192.png'))) }}" class="logo" alt="{{ $slot }}">
</a>
</td>
</tr>
```
