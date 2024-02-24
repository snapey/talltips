---
description: An alternative to Google Recaptcha
---

# Create a Sliding Puzzle Captcha

SlidingCaptcha is a simple class that creates a sliding puzzle.  The user must align the pieces when submitting a form such as a registration or contact form.  It does not rely on any third party API and satisfies privacy concerns.&#x20;

{% hint style="info" %}
Updated for Intervention V3, which requires PHP 8+
{% endhint %}

![](.gitbook/assets/SlidingCaptcha.gif)

The background to the puzzle is generated on-the-fly by Intervention image and passed to the view as inline images.

### SlidingCaptcha Service Class

Create a class called SlidingCaptcha.  Here I have created it in a `Services` folder

{% code title="SlidingCaptcha.php" %}
```php
<?php

namespace App\Services;

use Intervention\Image\Image;
use Intervention\Image\ImageManager;
use Intervention\Image\Drivers\Gd\Driver;
use Intervention\Image\Geometry\Factories\CircleFactory;

class SlidingCaptcha
{
    public $manager;

    public Image $top;

    public Image $bottom;

    public int $position;

    const CANVAS_HEIGHT = 200;

    const CANVAS_WIDTH = 4000;

    const CANVAS_BG = '#F0F0F0';

    public function __construct()
    {
        $this->manager = new ImageManager(new Driver());
        $this->generate();
    }

    private function generate()
    {
        $image = $this->createImage();

        $this->bottom = clone $image;
        $this->top = clone $image;

        $this->bottom->crop(2000, 50, 0, 50);

        $this->position = random_int(0, 160) * 10;  // ensures steps of 10

        $this->top->crop(400, 50, $this->position, 0);

        $this->position = 2000 - $this->position;

    }

    private function createImage()
    {
        $image = $this->manager->create(self::CANVAS_WIDTH, self::CANVAS_HEIGHT)->fill(self::CANVAS_BG);

        foreach (range(1, 50) as $x) {
            $image->drawCircle(
                random_int(0, self::CANVAS_WIDTH),   // x
                random_int(0, self::CANVAS_HEIGHT),  // y
                function (CircleFactory $circle) {
                    $circle->radius(random_int(20, (self::CANVAS_HEIGHT/2)-10)); // radius of circle in pixels
                    $circle->background($this->colours()); // background color
                    $circle->border('444444', 1); // border color & size
                });
        }

        $image->resize(self::CANVAS_WIDTH / 2, self::CANVAS_HEIGHT / 2);

        return $image;
    }

    private function colours()
    {
        return sprintf("rgba(%s, %s, %s, %s)",
            random_int(0, 255),  // range for R
            random_int(0, 255),  // range for G
            random_int(0, 255),  // range for B
            (rand(1, 8) / 10)    // range for opacity
        );
    }
}


```
{% endcode %}

Using the popular package Intervention Image ([https://image.intervention.io/v3](https://image.intervention.io/v3)) a canvas is created which is twice the size we need.  I found making it the exact size it was too grainy.  The canvas is initially 4000px x 200px and contains 50 randomly spaced and coloured circles.

The image is then downsized to 2000px x 100px, and then split into two halves, top and bottom. Finally, the top image is cropped to 400px wide at a random position within the larger image.

### Using the Service Class

Call the Service in the controller that presents the form;

```php
        $sc = new SlidingCaptcha();

        session()->put('sc_position', $sc->position);

        return view('test')->withSlidingCaptcha($sc);
```

Here we pass the SlidingCaptcha object to the view.  It contains two objects for both parts of the puzzle, and the position within the full image where the top image was taken from.  This will be what the user needs to provide by sliding the puzzle.

### The blade view

The view is very simple, and uses Tailwindcss for styling and Alpinejs to allow the user to slide the puzzle.

```markup
<form action="{{ route('contact.create') }}" method="POST"> @csrf
    <!-- rest of your form here -->
    <div class="flex flex-col" x-data="{guess:400}" x-effect="$refs.bottom.style.backgroundPosition=guess+'px';">
        <div class="w-full mx-auto rounded-t-md" style="margin:0; height:50px; background-image:url('{{ $slidingCaptcha->top->toGif()->toDataUri() }}')"></div>
        <div class="w-full mx-auto shadow rounded-b-md" style="margin:0; height:50px; background-image:url('{{ $slidingCaptcha->bottom->toGif()->toDataUri() }}');" x-ref="bottom" ></div>
        <input type="range" name="guess" min="400" max="2000" step="10" x-model="guess" autocomplete="off" class="w-full mt-2 py-3 max-w-[400px]">
        @error('guess'){{ $message }}@enderror
    </div>
    <input type="submit" value="Send" class="px-4 py-2 rounded-lg bg-emerald-600 font-bold shadow-lg text-white mt-4 border"> 
</form>
```

The top half of the puzzle and the bottom half are stacked on top of each other and then a range input element provides the amount that the bottom image should be scrolled by.

Alpine links the value of the input slider with the position of the background so as the slider moves, so does the bottom image within its container.

When the two halves align, the user can try submitting the form

### Validating the input

When we created the SlidingCaptcha, we saved the `position` in session so that we can check it when the form is submitted with simple validation which can be added alongside your other validation rules.

```php
    public function create(Request $request)
    {
        $this->validate($request, [
            'guess' => ['required', Rule::in([session('sc_position')])],
        ],[
            'guess.in' => 'The puzzle must be aligned exactly'
        ]);
```

The guess from the form (the range input element) must match exactly, the position that was stored in session.  If it does then the user passed the Captcha challenge!



