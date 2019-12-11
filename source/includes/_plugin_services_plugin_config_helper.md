### Plugin Config Helper
```php
<?php 
$configHelper = $this->get('caymland.helper.bundle');

$menu = $configHelper->getBundleConfig('HelloWorldBundle', 'menu', true);
```

* Service name: `caymland.helper.bundle`
* Class: `Caymland\CoreBundle\Helper\BundleHelper`

This can be used to get the configuration array of a bundle/plugin's Config/config.php file.
