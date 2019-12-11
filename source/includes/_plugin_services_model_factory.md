### Model Factory

```php
<?php

$channel   = 'email';
$channelId = 1;

if ($modelFactory->hasModel($channel)) {
    $model = $modelFactory->getModel($channel);
    
    if ($entity = $model->getEntity($channelId)) {
        echo $entity->getName();
    }
}
```

`Caymland/CoreBundle/Factory/ModelFactory` can be used in services that a model dependency is unknown at the time the service is created. This is great for scenarios where the channel and channel ID are stored in a database and the executing code needs to obtain information on the channel entity (name, etc). 

It has two methods: `hasModel($modelNameKey)` and `getModel($modelNameKey)`. `hasModel` simple checks to see if a model exists. It uses the same format as using the controller helper method `getModel()`. For example, to obtain the `Caymland\EmailBundle\Model\EmailModel` class, you could use something like the code example.
