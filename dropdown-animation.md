---
description: Dropdown menu effect using Tailwind and Alpine
---

# Dropdown animation

![](.gitbook/assets/dropdown.gif)

```markup
<div class="bg-gray-200 flex min-h-screen items-center ml-32">
  <div class="inline-block relative" x-data="{open: false}">
    <button @click="open = !open" class="focus:outline-none shadow cursor-pointer inline-block text-gray-700 hover:text-black flex border border-gray-400 rounded p-2 pl-3 pr-1 bg-gray-100" :class="{ 'shadow-none border-indigo-300': open}">
      @snapey
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" :class="{'rotate-180': open}" class="ml-1 transform duration-300 inline-block fill-current text-gray-500 w-6 h-6"><path fill-rule="evenodd" d="M15.3 10.3a1 1 0 011.4 1.4l-4 4a1 1 0 01-1.4 0l-4-4a1 1 0 011.4-1.4l3.3 3.29 3.3-3.3z"/></svg>
    </button>

    <ul x-show="open" class="bg-white absolute left-0 shadow w-40 rounded text-indigo-600 origin-top shadow-lg"
      x-transition:enter="transition ease-out duration-200"
      x-transition:enter-start="opacity-0 transform scale-y-50"
      x-transition:enter-end="opacity-100 transform scale-y-100"
      x-transition:leave="transition ease-in duration-300"
      x-transition:leave-end="opacity-0 transform scale-y-50"
    >
      <li><a href="#" class="py-1 px-3 block hover:bg-indigo-100">Profile</a></li>
      <li><a href="#" class="py-1 px-3 border-b block hover:bg-indigo-100">Billing</a></li>
      <li><a href="#" class="py-1 px-3 block hover:bg-indigo-100">Log out</a></li>
    </ul>
  </div>
</div>
```

Adapted from [https://www.jesper.dev/posts/creating-a-dropdown-with-alpinejs/](https://www.jesper.dev/posts/creating-a-dropdown-with-alpinejs/)
