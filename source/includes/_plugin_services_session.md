### Session

```php
<?php

$session        = $this->get('session');
$requestSession = $request->getSession(); // Shortcut to session

// Get all session parameters
$all = $session->all();

// Get specific parameter setting mars as default
$world = $session->get('helloworld.world', 'mars');

// Check if a parameter exists
if ($session->has('helloworld.world')) {
    // do something
}

// Set a session parameter
$session->set('helloworld.world', 'mars');

// Remove a session parameter
$session->remove('helloworld.world');

// Clear the whole session
$session->clear();
```

* Service name: `session`
* Class: `Symfony\Component\HttpFoundation\Session`
* Docs: [http://symfony.com/doc/2.8/components/http_foundation/sessions.html](http://symfony.com/doc/2.8/components/http_foundation/sessions.html)

To access the session service in a view, use `$app->getSession()`.