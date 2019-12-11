### Security

```php
<?php

$security = $this->get('caymland.security');

// Check if user is granted a single permission
if ($security->isGranted('plugin:helloWorld:worlds:view')) {
    // do something
}

// Check if user is granted multiple permissions (must be granted to all to be true)
if ($security->isGranted(
    array(
        'plugin:helloWorld:worlds:view',
        'plugin:helloWorld:worlds:create',
    )
)
) {
    //do something
}

// Check if user is granted to at least one permission
if ($security->isGranted(
    array(
        'plugin:helloWorld:worlds:view',
        'plugin:helloWorld:worlds:edit',
    ),
    'MATCH_ONE'
)
) {
    //do something
}

// Get an array of user permissions
$permissions = $security->isGranted(
    array(
        'plugin:helloWorld:worlds:view',
        'plugin:helloWorld:worlds:edit',
    ),
    'RETURN_ARRAY'
);

if ($permissions['plugin:helloWorld:worlds:view']) {
    // do something
}

// Check to see if a user is anonymous (not logged in)
if ($security->isAnonymous()) {
    // do something
}
```

* Service name: `caymland.security`
* Class: `Caymland\CoreBundle\Security\Permissions\CorePermissions`

Using the service to check permissions is explained more in [Using Permissions](#using-permissions).
