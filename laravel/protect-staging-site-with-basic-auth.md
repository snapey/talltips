---
description: When you don't want public sites crawling
---

# Protect Staging site with Basic Auth

If you have a staging instance of your website, it will probably look just like your production server and have all the same pages.

This can cause your production site to be penalised by search engines (because of duplicate content) or potential confusion by customers who happen upon your staging site when then actually want production.

A simple solution is to add basic authentication when the site's environment is \`staging\`.

### Create a middleware

I called this StagingBasicAuth, you can choose this name or whatever makes sense to you

{% code title="App\Http\Middleware\StagingBasicAuth.php" %}
````php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\App;
use Illuminate\Contracts\Auth\Factory as AuthFactory;


class StagingBasicAuth
{
        /**
         * The guard factory instance.
         *
         * @var \Illuminate\Contracts\Auth\Factory
         */
        protected $auth;
    
        /**
         * Create a new middleware instance.
         *
         * @param  \Illuminate\Contracts\Auth\Factory  $auth
         * @return void
         */
        public function __construct(AuthFactory $auth)
        {
            $this->auth = $auth;
        }
    
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string|null  $guard
         * @param  string|null  $field
         * @return mixed
         *
         * @throws \Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException
         */
        public function handle($request, Closure $next, $guard = null, $field = null)
        {
            if(App::environment() == 'staging') {
                $this->auth->guard($guard)->basic($field ?: 'email');
            }
            
            return $next($request);
        }
    }
    

```

````
{% endcode %}

If the app environment is NOT staging then the middleware is skipped and has no effect

### Add middleware to the 'web' group

Add the middleware to the array of `$middlewaregroups` (Laravel 10) / `$routeMiddleware` (earlier).

```php
            \App\Http\Middleware\StagingBasicAuth::class,
            
```

When `APP_ENV = staging` no content from the site will be accessible without first logging in.

### Setting Credentials

By default, the basic authentication guard will validate the user against the users table with the `email` field and hashed password.

### Clearing Basic Auth credentials in Chrome

Whilst testing this, you may come across an issue where Chrome (and possibly others) refuse to logout from the site since as soon as you access the site it sends the cached credentials.  The only reliable method I have found to clear the credentials is to add a route to your site like;

```php
Route::get('/clearbasic', function() { auth()->logout(); abort(401);});

```

Hit this route and Chrome will forget what it thinks are invalid credentials.  You can then revisit the site and be re-prompted for the login.

