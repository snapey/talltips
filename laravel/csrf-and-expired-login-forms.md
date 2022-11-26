---
description: How to handle a page with a login form that will expire
---

# CSRF and expired login forms

If your homepage contains a login form, or a modal with login, then when the session ends (by default, after 2 hours) then the csrf token is no longer valid and the user sees a page expired warning after they have filled out their login details.

We can work around this with a simple addition to the `<head>` of the main layout template.

```markup
<meta http-equiv="refresh" content="{{ config('session.lifetime') * 60 }}">
```

This simple line will [refresh](https://en.wikipedia.org/wiki/Meta\_refresh) the page when it gets to the end of the session. The refreshed page will have a new session and a new csrf token. This way, your login form is always valid.

If the user interacts with the site  and loads other pages then this refresh will never happen since the timeout is reset each time the page is loaded.

For logged in users, after the session lifetime the page will refresh and they will be returned to the same page, however they will no longer be authorised so will be redirected however the auth middleware is configured.

There is a very small chance that the user goes away and comes back after 1 hour 59 minutes and starts to fill out the login form, part of the way through the page refreshes.  This would be a very unlikely coincidence and the user will be no worse off than if the form was stale and failed after they pressed login.

{% hint style="info" %}
Note that this reload will cause some small additional view count in your analytics&#x20;
{% endhint %}
