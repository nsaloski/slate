### Request
```php
<?php

$request = $this->get('request_stack')->getCurrentRequest();

// $_GET
$get = $request->query->all();

// $_POST
$post = $request->request->all();

// $_COOKIE
$cookies = $request->cookies->all();

// $_SERVER
$server = $request->server->all();

// Headers
$headers = $request->headers->all();

// Attributes - custom parameters
$headers = $request->attributes->all();

// Check if a parameter exists
if ($request->request->has('hello')) {
    // do something
}

// Retrieve value of a specific parameter setting mars as default
$world = $request->query->get('world', 'mars');

// Set custom request value
$request->attributes->set('hello', 'world');

// Get the value of a nested array
$mars = $request->request->get('world[mars]', array(), true);
```

* Service name: `request_stack`
* Class: `Symfony\Component\HttpFoundation\RequestStack`
* Docs: [http://symfony.com/doc/2.8/book/service_container.html#book-container-request-stack](http://symfony.com/doc/2.8/book/service_container.html#book-container-request-stack)

There are multiple ways to obtain the request service.
 
If the controller is extending one of [Caymland's controllers](#controllers), it is already available via `$this->request`. Alternatively, Symfony will [auto-inject the request object](http://symfony.com/doc/2.8/book/controller.html#the-request-object-as-a-controller-argument) into the controller action method if the variable is type-hinted as `Symfony\Component\HttpFoundation\Request`.

For services, pass the `request_stack` service then use `$request = $requestStack->getCurrentRequest()`.

From within a view, use `$app->getRequest()`.
