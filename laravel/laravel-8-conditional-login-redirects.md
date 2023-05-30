---
description: >-
  How to provide different redirects at login when using Laravel Fortify and
  Jetstream
---

# Jetstream Login Conditional Redirect

#### ✅ Checked works with Laravel 10

Laravel 8 introduces [Fortify](https://github.com/laravel/fortify), a new back-end package for providing user authentication services. This represents a big departure from the **controller with traits** approach used in previous versions and has caused some concern that the authentication process is no longer customisable.

Of course, the maturity of the framework, and previous experiences would be unlikely to produce a framework where you could not override default behaviour.

Suppose we need to redirect the user as they are logging in based on some user attribute. With Fortify, how might this be possible?

The hooks that we require are bound into the container during the booting of the `Laravel\Fortify\FortifyServiceProvider`. Within our own code we can re-bind a different class where we will place our business logic.

### Create our own Login Response Class

1. Create a folder under app\Http called `Responses`
2. Create a file `LoginResponse.php`

```php
<?php

namespace App\Http\Responses;

use Illuminate\Support\Facades\Auth;
use Laravel\Fortify\Contracts\LoginResponse as LoginResponseContract;

class LoginResponse implements LoginResponseContract
{

    public function toResponse($request)
    {
        
        // below is the existing response
        // replace this with your own code
        // the user can be located with Auth facade
        
        return $request->wantsJson()
                    ? response()->json(['two_factor' => false])
                    : redirect()->intended(config('fortify.home'));
    }

}
```

For Example

```php
<?php

namespace App\Http\Responses;

use Illuminate\Support\Facades\Auth;
use Laravel\Fortify\Contracts\LoginResponse as LoginResponseContract;

class LoginResponse implements LoginResponseContract
{

    public function toResponse($request)
    {
        return $request->wantsJson()
                    ? response()->json(['two_factor' => false])
                    : redirect()->intended(
                        auth()->user()->is_admin ? route('admin.dashboard') : route('dashboard')
                    );
    }

}
```

### Make Laravel use our new Response Class

This new class now replaces the Singleton previously registered by Fortify.

Edit the `JetstreamServiceProvider` in your `app\Providers` folder;

In the boot method, add reference to your new response class. When login completes (and the user is actually Authenticated) then your new response will be called.

```php
    public function boot()
    {
        $this->configurePermissions();

        Jetstream::deleteUsersUsing(DeleteUser::class);

        // register new LoginResponse
        $this->app->singleton(
            \Laravel\Fortify\Contracts\LoginResponse::class,
            \App\Http\Responses\LoginResponse::class
        );
    }
```

### Two Factor Authentication

If you use 2FA with Jetstream, you will also need to catch the TwoFactorLoginResponse.  Use the same approach;

```php

        // register new TwofactorLoginResponse
        $this->app->singleton(
            \Laravel\Fortify\Contracts\TwoFactorLoginResponse::class,
            \App\Http\Responses\LoginResponse::class
        );
```

You can return the same response, or create an additional response if you want different behaviour for users that login using 2FA.
