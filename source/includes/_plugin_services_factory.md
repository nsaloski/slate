### Factory Service

`\Caymland\Factory\CaymlandFactory` is deprecated as of 2.0 and will be phased out through 2.x release cycles and completely removed in 3.0. Direct dependency injection into the services should be used instead where possible. 

For [controllers](#controllers), extend either `\Caymland\CoreBundle\Controller\CommonController` or `\Caymland\CoreBundle\Controller\FormController` and it will be available via `$this->factory` by default. Otherwise, obtain the factory from the service container via `$factory = $this->container->get('caymland.factory');`
  
For [models](#models), it will be available via `$this->factory` by default.
  
For custom [services](#services), pass 'caymland.factory' as an argument and CaymlandFactory will be passed into the __construct of the service.
