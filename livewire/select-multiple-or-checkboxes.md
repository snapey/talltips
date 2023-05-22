---
description: How to set the values from existing data
---

# Select Multiple or Checkboxes

I see many questions on Laracasts where the person seems confused about how to set the value of the select input or checkboxes when using Laravel Livewire.

The simple answer is that you don't need to worry if you set your component up right then Laravel will take care of it for you.

The component illustrates the use of both select (with "multiple" enabled), and checkboxes.&#x20;

<figure><img src="../.gitbook/assets/Screenshot 2022-11-26 at 16.16.33 (1).png" alt=""><figcaption></figcaption></figure>

The code for the component looks like

{% code title="Livewire/RolesTest.php (part)" lineNumbers="true" %}
```php
<?php

namespace App\Http\Livewire;

use App\Models\User;
use Livewire\Component;
use Spatie\Permission\Models\Role;

class RolesTest extends Component
{
    public int $user_id;
    public string $user_name;
    public array $userRoles;

    public function mount(User $user)
    {
        $this->user_id = $user->id;
        $this->user_name = $user->name;

        $this->userRoles = $user->roles()->pluck('id')->toArray();
    }

    public function render()
    {
        return view('livewire.roles-test')
            ->withRoles(
                cache()->remember('roles',60, function(){
                    return Role::all();
                })
            );
    }

    protected $rules = [
        'userRoles.*' => 'exists:roles,id',
    ];

    public function submit()
    {
        $this->validate();
 
        $user = User::findOrFail($this->user_id);

        $user->roles()->sync($this->userRoles);

     }

}

```
{% endcode %}

The important points are;

**Line 13,** the user's existing roles are pulled from the user model roles relationship, and stored in the component **as an array**

**Line 26,** all possible roles are passed to the view.  Here, the cache is used to keep a record of the possible options in the cache for 60 seconds. So if the component is changed multiple times, then the roles are not re-queried from the database each time.

**Line 34**, since we are holding the choices as an array, we can use Laravel's array validation to ensure that each member of the array matches the rules, and in this case, the role must exist in the roles table.

**Line 43**, once the roles have been chosen, since they are an array of IDs, they can be simply passed into the sync() function against the roles relationship.  No matter what the roles before the save, syncing the roles with the array will set the new roles only.

### The view

#### Select box with multiple option

The view contains both input types for the benefit of this article only.  I suggest that you use checkboxes over the multi-select because of the tricky UI to select multiple, and not accidentally clearing existing roles.

{% code title="roles-test.blade.php (part)" lineNumbers="true" %}
```html
    {{-- Method using multi-select input --}}
    <select multiple wire:model.lazy="userRoles" class="w-1/2 rounded form-multiselect">
        @foreach($roles as $role)
            <option value="{{$role->id}}">{{$role->name}}</option>
        @endforeach
    </select>
    <div class="text-sm italic text-gray-600">Alt/Cmd click to select multiple roles</div>
    
```
{% endcode %}

For the Multiple Select, we need to `wire:model` the select to the roles array then iterate over the possible roles for each option.  We do NOT need to use the `selected` html parameter on each option as this will be applied automatically by Livewire' `wire:model.`

Neither do we need to worry about the `old()` helper, again Livewire will take care of this for us.

#### The Checkbox Option

{% code lineNumbers="true" %}
```html
    {{-- method using checkboxes --}}
    <div class="flex flex-col my-8 space-y-1">
        @foreach($roles as $role)
            <div class="flex justify-between">
                <label for="role-{{$role->name}}">{{$role->name}}</label>
                <input class="rounded form-checkbox" id="role-{{$role->name}}" 
                    type="checkbox" value="{{$role->id}}" wire:model.lazy="userRoles" />
            </div>
        @endforeach
    </div>
```
{% endcode %}

For each of the possible roles, we output a label and checkbox. Each individual checkbox is bound to the array of the user roles. Livewire takes care of changing the correct array member according to the `value` of the checkbox.&#x20;

Again, we don't need to care about the `old()` helper or the `checked` state

## Beware of user manipulation of public component properties

With Livewire2, you should be wary of a user manipulating the public properties of your component, such as changing the user\_id to that of another user.  Check the video;

{% embed url="https://www.youtube.com/watch?v=bA1dMbUiwuA" %}

{% hint style="info" %}
**You can discuss pages on this site at** [**https://github.com/snapey/talltips/discussions**](https://github.com/snapey/talltips/discussions)
{% endhint %}

