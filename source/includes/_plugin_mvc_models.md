### Models

```php
<?php
// plugins/HelloWorldBundle/Model/ContactModel.php

namespace CaymlandPlugin\HelloWorldBundle\Model;

use Caymland\CoreBundle\Model\CommonModel;

class ContactModel extends CommonModel
{

    /**
     * Send contact email
     * 
     * @param array $data
     */
    public function sendContactEmail($data)
    {
        // Get mailer helper - pass the caymland.helper.mailer service as a dependency
        $mailer = $this->mailer;

        $mailer->message->addTo(
            $this->factory->getParameter('mailer_from_email')
        );

        $this->message->setFrom(
            array($data['email'] => $data['name'])
        );
        
        $mailer->message->setSubject($data['subject']);

        $mailer->message->setBody($data['message']);

        $mailer->send();
    }
}
```
Models are used to retrieve and process data between controllers and views. Models aren't required for plugins but, if used, Caymland provides means to easily obtain the model objects and some commonly used methods.

#### Model Classes

Model's should be registered as [`model` services](#service-types). The names of these services should match the following nomenclature: `caymland.UNIQUE_BUNDLE_IDENTIFIER.model.MODEL_IDENTIFIER`. `UNIQUE_BUNDLE_IDENTIFIER` can be whatever is desired but must be unique across all Caymland bundles and plugins. `MODEL_IDENTIFIER` just has to be unique for the given bundle. For example, the model example code could be registered as `caymland.helloworld.model.contact`. This allows the helper functions to retrieve model objects to find the correct model service. 

Custom models can extend one of two Caymland base models to leverage some helper functions:

##### \Caymland\CoreBundle\Model\AbstractCommonModel

This is the basic model that mainly provides access to services frequently used with models.

Property|Service
--------|-------
$this->factory | [Caymland's factory service](#factory-service) - deprecated as of 2.0; use direct dependency injection where possible
$this->em | [Entity manager service](#database)
$this->security | [Caymland's security service](#security)
$this->dispatcher | [Event dispatcher service](#events)
$this->translator | [Translator service](#translator)

##### \Caymland\CoreBundle\Model\FormModel

The FormModel extends AbstractCommonModel and provides a set of helper methods for interacting with entities and repositories. To read more about these methods, refer to the [Database](#database) section. 

#### Getting Model Objects

```php
<?php

/** @var \Caymland\LeadBundle\Model\LeadModel $leadModel */
$leadModel = $this->getModel('lead'); // shortcut for lead.lead

/** @var \Caymland\LeadBundle\Model\ListModel $leadListModel */
$leadListModel = $this->getModel('lead.list');

/** @var \CaymlandPlugin\HelloWorldBundle\Model\ContactModel $contactModel */
$contactModel = $this->getModel('helloworld.contact');
```

If using a model in another service or model, inject the model service as a dependency. If in a controller, use the `getModel()` helper function.
