---
description: Simple Laravel 7+ component to display one of the Tabler icon set
---

# Tabler Icons Component



{% embed url="https://github.com/tabler/tabler-icons" %}

## The Component

{% code title="resources/views/components/tabler.blade.php" %}
```markup
<!-- tabler icon, pulls from public/tabler folder. Accepts:
    strokeWidth   defaults to "1"
    class         defaults to "inline-block relative h-2"
-->

<svg viewBox="0 0 24 24" stroke="currentColor" 
    stroke-width="{{ $strokeWidth ?? 1 }}" 
    class="inline-block relative h-2 {{ $class ?? '' }}" 
    {{ $attributes->except(['class','icon']) }} 
>
    
    <use xlink:href="/tabler/tabler-sprite-nostroke.svg#tabler-{{$icon}}" />

</svg>
```
{% endcode %}

Copy the component script into `tabler.blade.php` in the `resources/views/components` folder

## Install the icons

Copy the icon svg sprite file from Github tabler/tabler-icons

Locate the file `tabler-sprite-nostroke.svg` and copy it into the folder  `public/tabler/`

## Usage

Example usage:

Twitch icon with height 6 in the same colour as the parent element.

```markup
<x-tabler icon="brand-twitch" class="h-6" />
```

Bucket icon in blue-500 and pushed up to align with the bottom of the text. Default stoke width of 1 is increased to 2.

```markup
<x-tabler wire:click="hello" 
    icon="bucket" 
    class="bottom-1 h-6 text-blue-500" 
    strokeWidth="2" />
```

#### Notes:

Search for the **icon name** required at [https://tablericons.com/](https://tablericons.com/)

Always remember to close the component with the `/>`&#x20;

If new icons are added just download a fresh copy of the svg file from Github. The project is under active development and new icons are being added regularly.

The approach uses the sprite map file for the icon set which is rather large at 300K+ . You may prefer to download the individual icon files and use the example above to find the correct file. At the time of writing, only individual icon files with 2px stroke weight are provided.



