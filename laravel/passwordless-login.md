---
description: Create a pass-phrase or 'magic-link' login system for Laravel 8 and Jetstream
---

# Password-less Login with Laravel 8

## Introduction

Users don't manage passwords well. They forget them or choose easy to remember passwords then use that same password on every site they visit.  We then have to build features into our application to let them login when the password is forgotten or allow them to change the password at any time. We have to ensure we hold passwords securely even if our application is not that important because a user might be trusting us with their use-everywhere password.

Our applications are easier to manage with less support issues if we use a password-less login process.  Such schemes are popular on sites like Medium with their 'magic link' or Notion.so with their login code such as `jay-bawl-sack-lid`

There is a good discussion of these password-less login methods in this [article on medium](https://medium.com/@kelvinvanamstel/should-we-embrace-magic-links-and-leave-passwords-alone-c73db7007fc4).

So, how could we implement such a solution with a new Laravel 8 Jetstream project?

{% hint style="info" %}
This article is using Jetstream with Livewire  (this site is TALL stack focussed) but the principles should hold for Inertia also.&#x20;
{% endhint %}

## Prepare

I'm starting with a new Laravel 8 project, with Jetstream installed in Livewire flavour.

## Remove the requirement for passwords

Our first task is to remove passwords from the Login process.  To make this simple, we will give everyone a default password of 'password'.  It won't be used, but it prevents us having to make too many changes to the Fortify+Jetstream code.

### &#x20;Remove password fields from login and register forms

{% code title="resources/views/auth/login.blade.php" %}
```markup
            <div>
                <x-jet-label value="Email" />
                <x-jet-input class="block w-full mt-1" type="email" name="email" :value="old('email')" required autofocus />
            </div>
{{-- 
            <div class="mt-4">
                <x-jet-label value="Password" />
                <x-jet-input class="block w-full mt-1" type="password" name="password" required autocomplete="current-password" />
            </div>
--}}
            <input type="hidden" value="password" name="password" />

```
{% endcode %}

{% hint style="info" %}
_Note the additional line for creating a hidden password field with the value 'password'_
{% endhint %}

{% code title="resources/views/auth/register.blade.php" %}
```markup
            </div>
{{-- 
            <div class="mt-4">
                <x-jet-label value="Password" />
                <x-jet-input class="block w-full mt-1" type="password" name="password" required autocomplete="new-password" />
            </div>

            <div class="mt-4">
                <x-jet-label value="Confirm Password" />
                <x-jet-input class="block w-full mt-1" type="password" name="password_confirmation" required autocomplete="new-password" />
            </div>
--}}

            <div class="flex items-center justify-end mt-4">
```
{% endcode %}

### Create user with default password

Fortify actions are available in the app/Actions/Fortify folder. We can adjust the CreateNewUser.php file to use our default password.

{% code title="app/Actions/Fortify/CreateNewUser.php" %}
```php
public function create(array $input)
{
    Validator::make($input, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
         // 'password' => $this->passwordRules(),
    ])->validate();

    return User::create([
        'name' => $input['name'],
        'email' => $input['email'],
        'password' => Hash::make('password'),    //make($input['password']),
    ]);
}
```
{% endcode %}

Here, the password field is commented out of the validation, and the password added to the user record is just the hash of the string 'password'.

**Testing:**  We should now be able to register a user, and login without any password.  _We have built a very insecure application at this point!_

![](../.gitbook/assets/sixtbanner.jpg)

_Article sponsored by_ [_SixTokens.com_](https://sixtokens.com)

## Create a source of pass-phrases

For this solution a passphrase is a combination of 3 or 4 words separated by hyphens. The words are sourced from a list[ published by the EFF](https://www.eff.org/deeplinks/2016/07/new-wordlists-random-passphrases) and are chosen because they are short and easy to spell.

Rather than publish the code and word list here, you can access it via [https://github.com/snapey/passphrase](https://github.com/snapey/passphrase)

* Create a folder within app called `Utility`
* Place **PassPhrase.php** and **wordlist.txt** in this folder

## Put pass-phrase into session when user logs in

When the user registers, Laravel will fire a `Registered` event and when the user is logged in either by the login form or the _remember_ function, then a `Login` event is fired.  We are going to listen for these events and place our randomly generated pass-phrase into session. Ultimately, the user will only be allowed access to our application if they can provide the same code that we have in session.

### Create a listener

`php artisan make:listener RequirePassPhrase`

This uses our utility class to create a pass-phrase and store it along with an expiry timestamp.

{% code title="app/Listeners/RequirePassPhrase.php" %}
```php
<?php

namespace App\Listeners;

use App\Utility\PassPhrase;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Session;

class RequirePassPhrase
{
    protected $generator;
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct(PassPhrase $generator)
    {
        $this->generator = $generator;
    }

    /**
     * Handle the event.
     *
     * @param  object  $event
     * @return void
     */
    public function handle($event)
    {
        // don't need to interrupt the process if the user 
        // logged in with remember token
        if(auth()->viaRemember()) {
            return;
        }

        $passphrase = $this->generator->passPhrase(3);

        Session::put('passphrase', $passphrase);
        Session::put('passphrase_expiry', now()->addMinutes(15)->timestamp);

    }
}

```
{% endcode %}

With this; `$this->generator->passPhrase(3);` we create a phrase with three words.  Set this according to your preferences.  The EFF article mentioned earlier explains;&#x20;

> for _k_ words chosen from a list of length _n_, there are _nk_ possible passphrases of this type. It will take an adversary about _nk_/2 guesses on average to crack this passphrase.

Our wordlist is approximately 4100 words, so 3 words is  4100x4100x4100/2 = 34,400,000,000 guesses so don't go overboard with the number of words in the passphrase.

### Bind our Listener to Events

Listening for events is configured in the app\Providers\EventServiceProvider.  We add our listener for the two events.  We can remove the email verification listener as if the user can receive the passcode then they verified the email at the same time.

{% code title="app/Providers/EventServiceProvider.php" %}
```php
    protected $listen = [
        \Illuminate\Auth\Events\Registered::class => [
            \App\Listeners\RequirePassPhrase::class,
        ],
        \Illuminate\Auth\Events\Login::class => [
            \App\Listeners\RequirePassPhrase::class,
        ],
    ];

```
{% endcode %}

## Send the pass-phrase to our user

Having created the code and put it in session we need to send this to the user.  Notifications are the easiest to implement here, and you could, for instance, choose to send the code to the user by one of the other notification methods such as SMS text.

### Create the notification

`php artisan make:notification AdvisePassPhrase`

Adjust your new notification (use your own words as required)

{% code title="app/Notifications/AdvisePassPhrase.php" %}
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;

class AdvisePassPhrase extends Notification
{
    use Queueable;

    public $passphrase;

    public function __construct(string $passphrase)
    {
        $this->passphrase = $passphrase;
    }

    public function via($notifiable)
    {
        return ['mail'];
    }

    public function toMail($notifiable)
    {
        return (new MailMessage)
            ->subject('Your login code for ' . config('app.name'))
            ->line('Here is your login PassPhrase which is valid for the next 15 minutes')
            ->line($this->passphrase)
            // ->action('Notification Action', url('/'))
            ->line('Thank you for using our application!');
    }

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            //
        ];
    }
}

```
{% endcode %}

### Call the Notification and provide the pass-phrase

In our earlier Listener, add a line to send the notification;

{% code title="app/Listeners/RequirePassPhrase.php" %}
```php
use App\Notifications\AdvisePassPhrase;


        Session::put('passphrase', $passphrase);
        Session::put('passphrase_expiry', now()->addMinutes(15)->timestamp);

        $event->user->notify(new AdvisePassPhrase($passphrase));
```
{% endcode %}

Line 7 is added to the earlier file

### Testing

Provided we have configured a mail service such as mailtrap, when we register or login, a mail should be received containing our passphrase.

## Accept and validate the pass-phrase

### Create Controller

`php artisan make:controller PassPhraseController`

{% code title="app/Http/Controllers/PassPhraseController.php" %}
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Session;
use Illuminate\Validation\ValidationException;
use Laravel\Fortify\Contracts\LoginResponse;

class PassPhraseController extends Controller
{
    public function show()
    {
        return view('auth.passphrase');
    }

    public function store(Request $request)
    {

        if (Session::get('passphrase_expiry') < now()->timestamp ){
            Auth::logout();
            $this->clearSession($request);
            return redirect()->route('login')->withErrors(['email' =>['Your Passphrase has expired. Please login again']]);
        }

        if (strToLower($request->passphrase) != Session::get('passphrase')) {
            throw ValidationException::withMessages([
                'passphrase' => ['Sorry, that is not the correct passphrase. Please check your email for the latest message.'],
            ]);
        }

        $this->clearSession($request);

        return app(LoginResponse::class);
    }

    public function clearSession($request)
    {
        $request->session()->forget('passphrase');
        $request->session()->forget('passphrase_expiry');
    }
}

```
{% endcode %}

Call this from your routes file

{% code title="routes/web.php" %}
```php
use App\Http\Controllers\PassPhraseController;

//

Route::get('/login/confirm',[PassPhraseController::class,'show'])->name('login.confirm');
Route::post('/login/confirm',[PassPhraseController::class,'store'])->name('login.confirmation');
```
{% endcode %}

### Create a form for the capture of the pass-phrase

The easiest route with a new application is to just copy the Login view and edit a few of the fields;

{% code title="resources/views/auth/passphrase.blade.php" %}
```php
<x-guest-layout>
    <x-authentication-card>
        <x-slot name="logo">
            <x-authentication-card-logo />
        </x-slot>

        <x-validation-errors class="mb-4" />

        @if (session('status'))
            <div class="mb-4 font-medium text-sm text-green-600">
                {{ session('status') }}
            </div>
        @endif

        <form method="POST" action="{{ route('login.confirmation') }}">
            @csrf

            <div>
                <x-label value="Pass-phrase" for="passphrase" />
                <x-input class="block w-full mt-1" type="text" name="passphrase" :value="old('passphrase')" required autofocus />
            </div>
            
            <div class="flex items-center justify-end mt-4">
                <x-button class="ml-4">
                     Confirm
                </x-button>
            </div>
        </form>
    </x-authentication-card>
</x-guest-layout>

```
{% endcode %}

### Testing

* Visit the route `/login/confirm` and check you can see the form.
* Entering an invalid code should show the message that the code is incorrect
* Entering a valid code should direct to the home route
* Waiting 15 minutes and entering a code should report that the code has expired

## Add middleware to block access until pass-phrase accepted

We can create a middleware that checks the user's session.  If it contains a `passphase` key then the user is in the middle of logging in and should not be permitted to access the application.  We need to except the login routes from the middleware so that the user can access the login process.

### Make Middleware

`php artisan make:middleware PassPhraseGuard`

{% code title="app/Http/Middleware/PassPhraseGuard.php" %}
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class PassPhraseGuard
{

    // if the user's session contains a passphrase then we need to direct the user to the 
    // passphrase confirm route instead.
    // need to allow the user through to any login routes

    public function handle(Request $request, Closure $next)
    {
        $passphrase = $request->session()->get('passphrase', null);

        if (is_null($passphrase)) {
            return $next($request);
        }

        // passphrase set, still valid?

        if ($request->session()->get('passphrase_expiry') < now()->timestamp) {

            $request->session()->forget('passphrase');
            $request->session()->forget('passphrase_expiry');

            Auth::logout();

            return redirect('/');
        }

        if ($request->route()->named('login.*')) {
            return $next($request);
        }

        return redirect()->route('login.confirm');
    }
}

```
{% endcode %}

### Include the Middleware as a global route middleware

Include the new middleware in your web middleware stack

{% code title="app/Http/Kernel.php" lineNumbers="true" %}
```php
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Laravel\Jetstream\Http\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            \App\Http\Middleware\PassPhraseGuard::class,
        ],
```
{% endcode %}

_We added Line 10_

### Testing

* Once logged in, access to all pages should be blocked, directing the user to the confirm pass-phrase page.
* Landing on the site after 15 minutes should return to the guest mode
* Remember me should work as normal

## Cleaning up

### Remove references to passwords

In the config/fortify.php file, turn off the ability to reset and change passwords by commenting out the `resetPasswords` and `updatePasswords` features.

{% code title="config/fortify.php" %}
```php

    'features' => [
        Features::registration(),
        // Features::resetPasswords(),
        // Features::emailVerification(),
        Features::updateProfileInformation(),
        // Features::updatePasswords(),
        Features::twoFactorAuthentication(),
    ],
```
{% endcode %}

## Conclusion

In this article we created a password-less login process that uses a **pass-phrase** technique. [ In the second part of this article we will add a **magic-link** alternative](password-less-login-with-magic-link-in-laravel-8.md).

## Feedback

If you have any suggestions how this article can be improved, DM the author on Twitter [@snapey](https://twitter.com/snapey)
