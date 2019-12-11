### Cookie Helper

```php
<?php

$cookieHelper = $this->get('caymland.helper.cookie');
$cookieHelper->setCookie('name', 'value', 3600);

$cookieHelper->deleteCookie('name');
```

* Service name: `caymland.helper.cookie`
* Class: `Caymland\CoreBundle\Helper\CookieHelper`

The cookie helper can be used to set cookies based on system settings.
