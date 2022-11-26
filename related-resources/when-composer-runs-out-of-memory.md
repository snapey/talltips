# When Composer runs out of memory

{% hint style="info" %}
Upgrade to Composer 2 to avoid memory issues
{% endhint %}

You can remove php memory restrictions with

`php -d memory_limit=-1 /usr/local/bin/composer require league/flysystem-aws-s3-v3`

Where `/usr/local/bin` is the path to your composer file.

You can check the path to composer with the command `which composer`

