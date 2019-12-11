### Translator

```php
<?php

$translator = $this->get('translator');
 
// Simple string
echo $translator->trans('plugin.helloworld.goodbye');

// Simple string with placeholders
echo $translator->trans('plugin.helloworld.greeting', array('%name%' => $name));

// String from a domain other than messages (will use planets.ini)
echo $translator->trans('plugin.helloworld.worlds', array('%world%' => $world), 'planets');

// Plural translations
$planetCount = 3;
echo $translator->transChoice('plugin.helloworld.number_of_planets', $planetCount, array('%planets%' => $planetCount));

// Check to see if a translation key exists
if ($translator->hasId('plugin.helloworld.goodbye')) {
    echo $translator->trans('plugin.helloworld.goodbye');
} else {
    // other logic
}

// Use the first key if it exists, otherwise use the second (helpful to prevent managing duplicate keys with the same string)
echo $translator->transConditional('plugin.helloworld.planets.' . $planet, 'plugin.helloworld.dwarf_planets. ' . $planet);
```

* Service name: `translator`
* Class: `Caymland\CoreBundle\Translation\Translator`
* Docs: [http://symfony.com/doc/2.8/components/translation/usage.html](http://symfony.com/doc/2.8/components/translation/usage.html)

Use the translator service to include translated strings in the code. Depending on where the translation is necessary will determine how to obtain the service.
 
To use the template service in view templates, simply use the [template helper](#translation-helper), `$view['translator']`.

The translator service has the following functions to help with translating strings:

**Simple translation**<br />
`trans($id, array $parameters = array(), $domain = null, $locale = null)`

**[Pluralization](http://symfony.com/doc/current/components/translation/usage.html#pluralization)**<br /> 
`transChoice($id, $number, array $parameters = array(), $domain = null, $locale = null)`

**Check to see if a key exists**<br />
`hasId($id, $domain = null, $locale = null)`

**Use the $preferred key if it exists, if not, use $alternative**<br />
`transConditional($preferred, $alternative, $parameters = array(), $domain = null, $locale = null)`
