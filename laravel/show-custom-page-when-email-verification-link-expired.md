---
description: Using Laravel 10 with Breeze + Blade starter kit
---

# Show custom page when email verification link expired

{% hint style="info" %}
For Breeze + Livewire + Alpine starter kit, see the section at the end
{% endhint %}

This is how you might change the default breeze installation to show a custom page when the signature in an account verification email has expired;

find the below in routes\auth.php

{% code title="routes\auth.web" %}
```php
    Route::get('verify-email/{id}/{hash}', VerifyEmailController::class)
                ->middleware(['signed', 'throttle:6,1'])
                ->name('verification.verify');
```
{% endcode %}

remove the `signed` middleware

```php
    Route::get('verify-email/{id}/{hash}', VerifyEmailController::class)
                ->middleware(['throttle:6,1'])
                ->name('verification.verify');
```

In the VerifyEmailController, add the following before the existing code;

```php
        if (! $request->hasValidSignature()) {
            return redirect()->route('invalidSignature');
        }
```

This moves the signature checking from the middleware into the controller so that we can take control of the response. Here we redirect the user to a new view of our own design.

Duplicate the file resources/views/auth/verify-email.blade into verify-invalid.blade.php

Change it as below (only the message in the first div changes);

```html
<x-guest-layout>
    <div class="mb-4 text-sm text-gray-600">
        {{ __('Sorry but that verification link is no longer valid, click below to request a new one.') }}
    </div>

    @if (session('status') == 'verification-link-sent')
        <div class="mb-4 text-sm font-medium text-green-600">
            {{ __('A new verification link has been sent to the email address you provided during registration.') }}
        </div>
    @endif

    <div class="flex items-center justify-between mt-4">
        <form method="POST" action="{{ route('verification.send') }}">
            @csrf

            <div>
                <x-primary-button>
                    {{ __('Resend Verification Email') }}
                </x-primary-button>
            </div>
        </form>

        <form method="POST" action="{{ route('logout') }}">
            @csrf

            <button type="submit" class="text-sm text-gray-600 underline rounded-md hover:text-gray-900 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
                {{ __('Log Out') }}
            </button>
        </form>
    </div>
</x-guest-layout>
```

Create a new route in auth.php

```php
    Route::view('verify-invalid', 'auth.verify-invalid')
                ->name('invalidSignature');
```

This adds a new route that can show our new page.

### For Breeze with Livewire+Alpine

The main difference to the above process is that the view to copy is located in resources/views/livewire/pages/auth/verify-email.blade.php



{% hint style="info" %}
Remember that the default behaviour is that the user must be logged in when checking the verification email since the routes are wrapped in auth middleware
{% endhint %}
