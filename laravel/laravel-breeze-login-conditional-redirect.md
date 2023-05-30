---
description: >-
  When you want to redirect a user after login according to their role, and
  using Laravel Breeze
---

# Laravel Breeze Login Conditional Redirect

#### ✅ Checked works with Laravel 10

When using Breeze and having users with different roles (eg customer / administrator), you might want to redirect the user once they have authenticated.

With Breeze this is fairly simple since the authentication process is performed in the user's App and can be easily modified.

Locate the file app/Http/Controllers/Auth/AuthenticatedSessionController.php

Replace the last line of the `store()` method with your redirect logic. For example;

```php
    public function store(LoginRequest $request): RedirectResponse
    {
        $request->authenticate();

        $request->session()->regenerate();

        //return redirect()->intended(RouteServiceProvider::HOME);

        return redirect()->intended(
            auth()->user()->is_admin ? route('admin.dashboard') : route('dashboard')
        );
    }
```

In the example a ternary is used, testing the is\_admin flag on the logged in user.  Be sure to retain the \`intended()\` function since this serves a valuable purpose.

### Intended route

The intended route is the name given to the place the user was trying to reach when they had to login.  Imagine the situation; the user is logged in and looking at your application's dashboard or similar.  They go away for a few hours then return and click on a menu item.

Since they are no longer logged in because their session expired, they can't go straight to the link and are redirected to the login page.  The route that they were trying to reach is stored in session as the `intended` route.  After logging in, if the session contains an intended route then they should be sent there.

The `intended()` function takes one parameter and this is a fallback route for when intended is not set.

When modifying the behaviour after login, its important to honor the intended route.

### Fortify + Jetstream

Performing the same redirect on Jetstream is a little more involved.  Check the following TallTips article: [laravel-8-conditional-login-redirects.md](laravel-8-conditional-login-redirects.md "mention")
