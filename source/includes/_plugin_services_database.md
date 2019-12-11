### Database/Entity Manager
```php
<?php
// From controller
$em         = $this->getDoctrine()->getManager();
$repository = $em->getRepository('HelloWorldBundle:World');
$worlds     = $repository->getEntities();

/** @var \CaymlandPlugin\HelloWorldBundle\Entity\World $world */
foreach ($worlds as $world) {
    $world->upVisitCount();
}

$repository->saveEntities($worlds);
```

Doctrine includes an ORM and DBAL layers. 

ORM/entity manager:

* Service name: `doctrine.orm.default_entity_manager`
* Class: `Doctrine\ORM\EntityManager`

DBAL (direct DB driver):

* Service name: `doctrine.dbal.connection`
* Class: `Doctrine\DBAL\Connection`

The entity manager can be used to interact with the bundle's repositories and entities. See [Database](##database) for more info. 
