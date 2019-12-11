### Event Dispatcher
```php
<?php

$dispatcher = $this->get('event_dispatcher');
if ($dispatcher->hasListeners(HelloWorldEvents::ARMAGEDDON)) {
    $event = $dispatcher->dispatch(HelloWorldEvents::ARMAGEDDON, new ArmageddonEvent($world));
    
    if ($event->shouldPanic()) {
        throw new \Exception("Run for the hills!");
    }
}
```

* Service name: `event_dispatcher`
* Class: `Symfony\Component\EventDispatcher\EventDispatcher`
* Implements `Symfony\Component\EventDispatcher\EventDispatcherInterface` (When type hinting, use this class since different environments may use different classes for the dispatcher)
* Docs: [http://symfony.com/doc/2.8/components/event_dispatcher/introduction.html#dispatch-the-event](http://symfony.com/doc/2.8/components/event_dispatcher/introduction.html#dispatch-the-event)

Dispatch [custom events](#custom-events) using the `event_dispatcher` service.
