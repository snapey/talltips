---
description: Reusing the same form for create and update
---

# Simplify Laravel CRUD Controllers

This is the pattern I use for simple CRUD operations. It makes use of Route Model Binding to inject the model being created, and model properties to determine if the model is in the process of being created or updated.

The forms also make use of the `old()` helper to pull in previous form value from the model, or from previously submitted form.

Note that the form elements here are styled with tailwind utility classes. If you are not into Tailwind, look past that as its not relevant to this article.

The example is CRUD for something called a Template. In this particular application, its just a form with a bunch of input fields.

## The Controller&#x20;

It starts with simplifying the controller;

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\TemplateForm;
use App\Template;
use Illuminate\Http\Request;

class TemplateController extends Controller
{

    public function create()
    {
        return $this->edit(new Template());
    }

    public function store(TemplateForm $request)
    {
        return $this->update($request, new Template());
    }

    public function edit(Template $template)
    {
        return view('template.edit')->withTemplate($template);
    }

    public function update(TemplateForm $request, Template $template)
    {
        $request->persist($template);

        return redirect(route('templates.index'));
    }

}
```

When creating a record, we create a new model and pass it into the edit function. The edit is responsible for returning the view. Because we pass a new model into edit, we don’t need to worry about how we use the model in the view (more later).

When the data is returned from the form, if it is a new model then again, we create a new instance of the Template model and pass it to the Update method. The Update does not care if the model is new or an existing one looked up by Route Model binding. All it needs to do is to pass the model back to the Form Request and ask it to persist the model with the form data.

## Sharing the form

The same form is shared for both edit and update functions. This is possible because either way, an instance of our model is passed to the form.

_I have abbreviated the form because its not relevant to the discussion, but you will see validation and persist for items that are not visible below._

```php
<div class="w-full p-6 flex">
    @if($template->exists)
        <form class="flex flex-col w-full" method="POST" action="{{ route('templates.update',$template) }}">
            @method('put')
    @else
        <form class="flex flex-col w-full" method="POST" action="{{ route('templates.store') }}">
    @endif
            @csrf
            <div class="flex w-full">
                {{-- form input element --}}
                <div class="flex flex-wrap mb-6 w-1/3">
                    <label for="name" class="block text-gray-700 text-sm font-bold mb-2">Template Name:</label>

                    <input id="name" type="text" required name="name"
                        value="{{ old('name', $template->name) }}"
                        class="text-base font-mono shadow appearance-none border rounded 
                            w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline 
                            @error('name') border-red-500 @enderror">
                    @error('name')
                    <p class="text-red-500 text-xs italic mt-4">{{ $message }}</p>
                    @enderror
                </div>

                {{-- form input element --}}
                <div class="flex flex-wrap mb-6 w-2/3 ml-4">
                    <label for="description" class="block text-gray-700 text-sm font-bold mb-2">Description:</label>

                    <input id="description" type="text" required name="description" value="{{ old('description', $template->description) }}"
                        class="text-base font-mono shadow appearance-none border rounded w-full 
                        py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline 
                        @error('description') border-red-500 @enderror">
                        
                    @error('description')
                    <p class="text-red-500 text-xs italic mt-4">{{ $message }}</p>
                    @enderror
                </div>
            </div>

            // irrelevant form elements removed.....

            <button class="positive-button" type="submit">Save </button>
        <form>
</div>
```

It is not possible to use the same form tag for both update and create because we need to pass the model ID for an update and make it a PUT request rather than a POST request. In line 2 we are checking if our model actually exists in the database so that we know which case it is. After this @if @else section, the rest of the form does not care if the model is new or not.

The form inputs themselves use the [`old()`](https://laravel.com/docs/6.x/requests#old-input)helper to insert the previous value, the value from validation or an empty value. for instance `value="{{ old('name', $template->name) }}"` . It helps a lot if you name the form field the same as the model attribute.

`old()` takes two parameters, the first is the field name that was submitted previously (in the case of validation failures, the second parameter is the default value. In our case, for a model that is being edited, the previous value is inserted. If it is a new model then NULL is returned and no errors are produced. First time around the value of the form field will be empty.

If you want a default value for the field then the null coalesce operator ?? can be used. For instance; `value="{{ old('type', $template->type ?? 'banana') }}"`. If the model is new then the default value of ‘banana’ will be inserted into the form.

## Form Request

I appreciate that this will be controversial, but it_'_s what I do, and is optional. You can still go ahead and store the updated model in the controller, or use repository pattern or whatever. I prefer to use the [form request class](https://laravel.com/docs/7.x/validation#form-request-validation) to save the form data also.

```php
<?php

namespace App\Http\Requests;

use App\Template;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Support\Facades\Storage;

class TemplateForm extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'name' => 'required|max:100',
            'description' => 'required|max:200',
            'subject' => 'max:200',
        ];
    }

    public function persist(Template $template)
    {
        $template->name = $this->name;
        $template->description = $this->description;
        $template->type = $this->type;
        $template->subject = $this->subject;
        $template->email_template = $this->email_template;
        $template->sms_template = $this->sms_template;

        $template->save();
    }
}
```

So, yes `rules()` is standard, but I have added a `persist()` method. This expects to be passed a model instance, to which it saves the form data.

The model instance was passed from the update function of the controller with `$request->persist($template);` If we are creating a model then an empty model was passed from the store method, and if we are updating an existing model then this was injected by [Route Model Binding](https://laravel.com/docs/6.x/routing#route-model-binding). In the form request class we just pop the values into the model and save it.

## Conclusion

Unfortunately, too many people think that if you want to bind model data to a form then you must use the Laravel Collective Form components :zany\_face: . This is not the case. Understanding the [old()](https://laravel.com/docs/7.x/helpers#method-old) helper is fundamental to building simple crud operations , and passing an **instance of a new model** to your form means that you can share the same form with create and update operations.
