### IP Lookup Helper
```php
<?php 
$ipHelper = $this->get('caymland.helper.ip_lookup');

$requestIp = $ipHelper->getIpAddressFromRequest(); // 1.2.3.4

/** @var \Caymland\CoreBundle\Entity\IpAddress $ipAddressEntity */
$ipAddressEntity = $ipHelper->getIpAddress();

/** @var array $details */
$details = $ipAddressEntity->getIpDetails();

echo $details['city'];
```

* Service name: `caymland.helper.ip_lookup`
* Class: `Caymland\CoreBundle\Helper\IpLookupHelper`

This helper can be used to retrieve the real IP for the request.
