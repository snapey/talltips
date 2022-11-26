---
description: Various tips for working with S3 storage from your Laravel application
---

# Using S3

See also;

{% content-ref url="../livewire/livewire-file-uploads-using-s3.md" %}
[livewire-file-uploads-using-s3.md](../livewire/livewire-file-uploads-using-s3.md)
{% endcontent-ref %}

## Content in folders

Segregate your S3 content according to your environment. Adding the following to `config/filesystems.php`

```php
        'pdfs' => [
            'driver' => 's3',
            'root' => env('APP_NAME') . '/pdfs',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION'),
            'bucket' => 'mybucket',
            'visibility' => 'public',

        ],
```

In the above, PDF files will be stored for public consumption within a disk prefixed with the environment name. The key to this is the `root` attribute. This provides a path to prefix to all files that are created in the pdfs disk.

Write files to the pdf folder after specifying the disk;

```php
Storage::disk('pdfs')->put('hello.txt','hello');
```

The `->url()` command can be used to get the public url for the created file;

```php
>>> Storage::disk('pdfs')->url('hello.txt');
=> "https://mybucket.s3.eu-west-1.amazonaws.com/MYAPP-DEV/pdfs/hello.txt"
```

Be sure to have different app names between Dev, Staging and Live to separate your output.

