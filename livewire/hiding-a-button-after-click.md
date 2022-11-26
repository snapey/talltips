---
description: >-
  Livewire can handle loading states in the browser to provide responsive
  feedback
---

# Hiding a button after click

You can hide a button, and reveal alternate text or spinner when the button is clicked

```markup
<button wire:loading.remove wire:target="send" wire:click="send" class="px-4 mt-2">Send Message</button>
<span wire:loading wire:target="send" class="inline-block px-4 my-3 font-bold text-red-700">Sending</span>
```

When the button is pressed, the `wire:loading.remove` will immediately hide the button until the request is complete. The target is itself.  At the same time the span is revealed with the same `wire:loading` and `wire:target` directives. Once the request is complete, the two elements are swapped back.

Use this for long requests (such as sending an email) to prevent multiple requests from the user impatiently pressing the button.
