## Plugin Directory Structure

```php
<?php
// plugins/HelloWorldBundle/HelloWorldBundle.php

namespace CaymlandPlugin\HelloWorldBundle;

use Caymland\PluginBundle\Bundle\PluginBundleBase;

class HelloWorldBundle extends PluginBundleBase
{
    // Nothing more required
}

```

The directory structure of an plugin will vary based on the features implemented. 

At a minimum, the following structure is required:

HelloWorldBundle/<br />
- - - Config/<br />
- - - - - config.php<br />
- - - HelloWorldBundle.php

Read more about the [config file](#plugin-config-file).

The HelloWorldBundle.php file registers the bundle with Symfony. See the code block for the minimum required code.

A typical MVC plugin may look something like:

HelloWorldBundle/<br />
- - - [Assets/](#asset-helper)<br />
- - - - - - images/ <br />
- - - - - - - - - earth.png<br />
- - - - - - - - - mars.png<br />
- - - - - - helloworld.js<br />
- - - - - - helloworld.css<br />
- - - [Config/](#plugin-config-file)<br />
- - - - - - config.php<br />
- - - [Controller/](#controllers)<br />
- - - - - - DefaultController.php<br />
- - - [Model/](#models)<br />
- - - - - - ContactModel.php<br />
- - - - - - WorldModel.php<br />
- - - [Translations/](#translations)/<br />
- - - - - - en_US/<br />
- - - - - - - - - flashes.ini<br />
- - - - - - - - - messages.ini<br />
- - - [Views/](#views)<br />
- - - - - - Contact/<br />
- - - - - - - - - form.html.php<br />
- - - - - - World/<br />
- - - - - - - - - index.html.php<br />
- - - - - - - - - list.html.php<br />
- - - HelloWorldBundle.php<br />
- - - HelloWorldEvents.php

Each of the other directories and files are explained elsewhere in the document. 
