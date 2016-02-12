# Generating links to Backend Modules

Since TYPO3 7.6, routing definitions can be placed in `EXT:Configuration/Backend/Routes.php`.

The old `tce_db.php`, `alt_doc.php` etc routes are no longer valid. Generating a route is 
now possible with `BackendUtility::getModuleUrl()`.

## Example URLs:
* `BackendUtility::getModuleUrl('tce_db')`,
* `BackendUtility::getModuleUrl('tce_file')`
* `BackendUtility::getModuleUrl('record_edit')`

## Example Routes.php

Taken from sysext backend.

```php
use TYPO3\CMS\Backend\Controller;
return [
  // Login screen of the TYPO3 Backend
  'login' => [
    'path' => '/login',
    'access' => 'public',
    'target' => Controller\LoginController::class . '::formAction'
  ]
];
```
