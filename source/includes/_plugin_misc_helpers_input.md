#### Input Helper

The input helper can be used to ensure clean strings from user input.

```php
<?php

use \Caymland\CoreBundle\Helper\InputHelper;

// ...

$clean = InputHelper::clean($input);
$clean = InputHelper::int($input);
$clean = InputHelper::alphanum($input);
$clean = InputHelper::html($input);

// and others; refer to the class for more options
```
<div class="clear-right"></div>
