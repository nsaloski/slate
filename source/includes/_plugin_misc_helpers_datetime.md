#### Date/Time Helper

The date/time helper can be used to convert between UTC and the local timezone. 

```php
<?php

$dtHelper  = new \Caymland\CoreBundle\Helper\DateTimeHelper('2015-07-20 21:39:00', 'Y-m-d H:i:m', 'local');
$utcString = $dtHelper->toUtcString();

// refer to the class for other functions
```
<div class="clear-right"></div>
