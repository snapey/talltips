---
description: When not using MIX and its versions
---

# CSS Cache Busting with your Git Commit SHA

Now that Tailwind CSS has JIT mode, I have stopped configuring and using laravel mix.

A useful feature of mix is cache busting - the ability to force your users to reload css and js files from the server instead of using their cached versions

A simple way to add cache busting to your css loads is to use your Git commit sha hash as a parameter on the css line.

This article describes how you can get the Git sha hash in your application; [https://talltips.novate.co.uk/laravel/versioning-your-laravel-project](https://talltips.novate.co.uk/laravel/versioning-your-laravel-project)

Add the sha to your css load;

```markup
<link rel="stylesheet" href="/css/app.css?{{ config('version.hash') }}">
```

This generates a line in your browser like;

```markup
<link rel="stylesheet" href="/css/app.css?f7509a3">
```

where \`f7509a3\` is the sha hash of your code and will change on each commit.
