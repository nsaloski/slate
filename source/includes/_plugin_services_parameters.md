### Config Parameters
```php
<?php 
$config = $this->get('caymland.helper.core_parameters');

$apiEnabled = $config->getParameter('helloworld_api_enabled', false);
```

* Service name: `caymland.helper.core_parameters`
* Class: `Caymland\CoreBundle\Helper\CoreParametersHelper`
