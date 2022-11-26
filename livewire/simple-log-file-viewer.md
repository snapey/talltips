---
description: Display log files in your application using Livewire and Alpine
---

# Simple Log File Viewer

I was having a lot of problems displaying log files in the admin area of my application.  The Logfiles were being written to following each transaction, and a typical daily log file could be up to 10MB. The best available packages would crash and burn with this size of logfile.

I tried both [rap2hpoutre/laravel-log-viewer](https://github.com/rap2hpoutre/laravel-log-viewer) and [arcanedev/log-viewer](https://github.com/ARCANEDEV/LogViewer) and ended up building a simple viewer with pagination of the logs. Alpine is used to collapse stack trace.

In TALL stack style, the logfiles are displayed using Tailwind.

### Livewire Component

`php artisan make:livewire LogsViewer`

{% code title="app/Http/Livewire/LogsViewer.php" %}
```php
<?php

namespace App\Http\Livewire;

use Illuminate\Support\Facades\File;
use Livewire\Component;
use SplFileInfo;

class LogsViewer extends Component
{
    public $file=0;
    public $page=1;
    public $total;
    public $perPage = 500;
    public $paginator;

    protected $queryString=['page'];

    public function render()
    {

        $files = $this->getLogfiles();

        $log=collect(file($files[$this->file]->getPathname(), FILE_IGNORE_NEW_LINES));

        $this->total = intval(floor($log->count() / $this->perPage)) + 1;

        $log = $log->slice(($this->page - 1) * $this->perPage, $this->perPage)->values();

        return view('livewire.logs-viewer')->withFiles($files)->withLog($log);


    }

    protected function getLogFiles()
    {
        $directory = storage_path('logs');

        return collect(File::allFiles($directory))
            ->sortByDesc(function (SplFileInfo $file) {
                return $file->getMTime();
            })->values();
    }

    public function goto($page)
    {
        $this->page=$page;
    }

    public function updatingFile()
    {
        $this->page=1;
    }
}

```
{% endcode %}

### View file

Of note here is the detection of the \[stackdump] sections, moving these to a nested block and hiding that block using Alpine. Clicking the `[stackdump]` in the log expands the nested section and reveals the stack dump.

{% code title="resources/views/livewire/logs-viewer.blade.php" %}
```php
<div>
    <x-slot name="header">
        <h2 class="text-xl font-semibold leading-tight text-gray-800">
            Log Files
        </h2>
    </x-slot>

    <div class="px-4 py-2 mx-4 my-8 bg-white shadow-xl sm:rounded-lg">
        <div class="flex justify-around">
            <select wire:model="file" class="px-4 py-2 font-mono text-sm bg-red-200 rounded">
                @foreach($files as $file)
                <option value="{{ $loop->index }}">{{ $file->getFilename() }}</option>
                @endforeach
            </select>
        </div>

        @include('layouts.logs-paginator')
        
        @if($log->count()>0)
            <ul class='font-mono text-xs'>
                
                @for($i=0; $i < $log->count(); $i++)
                    @if(Illuminate\Support\Str::startsWith($log[$i],'[stacktrace]') || Illuminate\Support\Str::startsWith($log[$i],'#'))
                        <li x-data="{expanded:false}" x-on:click="expanded = !expanded">[stacktrace]
                            <ul class="ml-8" x-show="expanded" x-cloak >
                                @while($i < $log->count())
                                    <li wire:key="{{$page}}-line-{{ $i }}">{{ $log[$i] }}</li>
                                    @break(Illuminate\Support\Str::startsWith($log[$i++],'"}'))
                                @endwhile
                            </ul>
                        </li>
                    @endif
                    @break($i>=$log->count())
                    
                    <li wire:key="{{ $page }}-line-{{ $i }}" class="font-mono text-xs leading-5  
                        {{ Illuminate\Support\Str::contains($log[$i], '.CRITICAL:') ? 'text-red-800':''}}
                        {{ Illuminate\Support\Str::contains($log[$i], '.ERROR:') ? 'text-orange-600':'' }}
                        {{ Illuminate\Support\Str::contains($log[$i], '.INFO:') ? 'text-blue-900':'' }}
                        {{ Illuminate\Support\Str::contains($log[$i], '.WARNING:') ? 'text-indigo-700':'' }}
                        ">{{ $log[$i] }}
                    </li>
                @endfor
            </ul>
        @endif
    </div>

    
</div>

```
{% endcode %}

### Paginator

{% code title="resources/views/layouts/logs-paginator.blade.php" %}
```php
<div class="flex float-right">
    <button id="first" wire:click="goto(1)" class="w-10 outline-none px-2 border rounded-l-lg m-0 {{ $page==1 ? 'bg-gray-600 text-white font-bold' :'' }}" >1</button>
    
    @if($page-4 > 1)
        <button id="dots1" class="w-10 px-2 m-0 -ml-px border outline-none">&hellip;</button>
    @endif

    @if($page-3 > 1)
        <button id="minus3" class="w-10 px-2 m-0 -ml-px border outline-none" wire:click="goto({{ $page-3 }})">{{ $page-3 }}</button>
    @endif
    @if($page-2 > 1)
        <button id="minus2" class="w-10 px-2 m-0 -ml-px border outline-none" wire:click="goto({{ $page-2 }})">{{ $page-2 }}</button>
    @endif
    @if($page-1 > 1)
        <button id="minus1" class="w-10 px-2 m-0 -ml-px border outline-none" wire:click="goto({{ $page-1 }})">{{ $page-1 }}</button>
    @endif

    @if($page != 1 && $page != $total )
        <button id="current" class="w-10 px-2 m-0 -ml-px font-bold text-white bg-gray-600 border outline-none">{{ $page }}</button>
    @endif

    @if($page+1 < $total )
        <button id="plus1" class="w-10 px-2 m-0 -ml-px border outline-none" wire:click="goto({{ $page+1 }})">{{ $page+1 }}</button>
    @endif
    @if($page+2 < $total )
        <button id="plus2" class="w-10 px-2 m-0 -ml-px border outline-none" wire:click="goto({{ $page+2 }})">{{ $page+2 }}</button>
    @endif
    @if($page+3 < $total )
        <button id="plus3" class="w-10 px-2 m-0 -ml-px border outline-none" wire:click="goto({{ $page+3 }})">{{ $page+3 }}</button>
    @endif
    
    @if($page+4 < $total )
        <button id="dots2" class="w-10 px-2 m-0 -ml-px border outline-none">&hellip;</button></button>
    @endif
    
    @if($total>1)
        <button id="last" class="rounded-r-lg w-10 outline-none px-2 border -ml-px m-0 {{ $page == $total ? 'text-white bg-gray-600 font-bold':'' }}" wire:click="goto({{ $total }})">{{ $total }}</button>
    @endif

</div>
```
{% endcode %}

Add a protected link to your `web.php` file to the Logfile component.

