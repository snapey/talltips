# Naming Livewire Components

If you put your livewire component in a sub folder then its name will be lowercase foldername followed by a period; eg

Livewire model of App\Http\Livewire\Modals\Notifications.php

```
@livewire('modals.notifications')

onclick="Livewire.emit('openModal', 'modals.notifications')" />
```

If your Component Class is Pascal Case, eg **DatabaseNotifications** then the class name called from the frontend will be lowercase and with a hyphen before any uppercase, eg

```
@livewire('modals.database-notifications')

onclick="Livewire.emit('openModal', 'modals.database-notifications')" />
```
