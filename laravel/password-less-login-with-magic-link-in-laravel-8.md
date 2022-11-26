---
description: Provide users with a secure URL that automatically logs them in
---

# Password-less Login with Magic Link in Laravel 8

This article follows on from code we built in [Password-less Login with Laravel 8](passwordless-login.md). Be sure to read that first as it requires much of what we scaffolded there.

## Send the user an email containing a secure url

In the AdvisePassPhrase notification we are also going to add back the button that we commented out earlier and attach our secure URL to the button.

{% code title="app/Notifications/AdvisePassPhrase.php" %}
```php
use Illuminate\Support\Facades\URL;

//

        public function toMail($notifiable)
    {
        return (new MailMessage)
            ->line('Here is your login PassPhrase which is valid for the next 15 minutes')
            ->line($this->passphrase)
            ->line('Or click the button to access the site (opens in a new window)')
            ->action('Confirm', URL::temporarySignedRoute(
                'login.magiclink',
                now()->addMinutes(15),
                ['user' => $notifiable->id, 'code' => $this->passphrase]
            ))
            ->line('Thank you for using our application!');
    }
```
{% endcode %}

* Line 11 we create our button labelled 'Confirm' and pass it a temporary signed route.
* Line 12 is the named route that we will create in a moment where we check the signed route.
* Line 13 we set the expiry time as 15 minutes in the future.&#x20;
* Line 14, we are passing and securing the user ID and the passphrase

## Create a controller for the secure endpoint

`php artisan make:controller MagicLinkController`

This controller only needs one function. Its responsibility is to check the URL is still secure and has not been tampered with and then clear the session passphrase keys.

{% code title="app/Http/Controllers/MagicLinkController.php" %}
```php
use App\Models\User;
use Illuminate\Support\Facades\Auth;

//

    public function confirm(Request $request, User $user)
    {
        if (!$request->hasValidSignature() || Auth::guest()) {
            abort(401);
        }
        
        if ($request->code != $request->session()->get('passphrase')) {
            abort(401);
        }

        $request->session()->forget('passphrase');
        $request->session()->forget('passphrase_expiry');

        return app(LoginResponse::class);
    }
```
{% endcode %}

If the signature is valid and the user is still logged in then clear the passphrase from session and perform the default LoginResponse.

If the Link has already been used then show an error.

{% hint style="info" %}
This process will not work if the user completes the login form on one device but then clicks the link in a different device. Since they are not logged in on that device this URL will have no effect. You might want to add this as a warning to the email.
{% endhint %}

## &#x20;Add the route

```php
Route::get('/login/magic/{user}',[MagicLinkController::class,'confirm'])->name('login.magiclink');
```

Make sure the route name starts `login.` so that it can bypass the protection we placed in our earlier middleware.

## Testing

When we login and provide the email address, an email is sent containing a secure link.  Clicking this link within 15 minutes should remove the passphrase we are using as a guard in middleware.

Any longer than 15 minutes and they will see an error.

## Feedback

If you have any suggestions how this article can be improved, DM the author on Twitter [@snapey](https://twitter.com/snapey)

