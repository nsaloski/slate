### Extending Webhooks

Webhooks allow for Caymland to send data to external services via an endpoint url.

Webhooks work by using event listeners for two main purposes:

1. An event listener to add webhook types to the webhook user interface.
2. An event listener to whichever action you want to trigger a webhook payload to be queued.

##### Additional Steps
Additional steps that you must take in your own bundle are:

1. Create your own event dispatching for your bundle's custom event and payload
2. Register your custom event listeners in your bundle's configuration file as a service.
3. Refer to [receiving webhooks](#receiving-webhook-payloads) section to receive payloads in your application.

#### Webhook Type Listener
<aside class="notice">
Use the <code>WebhookEvents::WebhookBuilderEvent</code> to add a webhook type to the webhook interface.
</aside>


```php
<?php
namespace Caymland\YourBundle\EventListener;

use Caymland\WebhookBundle\WebhookEvents;
use Caymland\WebhookBundle\Event\WebhookBuilderEvent;
use Caymland\CoreBundle\EventListener\CommonSubscriber;
use Caymland\YourBundle\YourBundleEvents;

/**
 * Class WebhookSubscriber
 *
 * @package Caymland\YourBundle\EventListener
 */

class WebhookSubscriber extends CommonSubscriber
{
    /**
     * @return array
     */
    static public function getSubscribedEvents()
    {
        return array(
            WebhookEvents::WEBHOOK_ON_BUILD => array('onWebhookBuild', 0)
        );
    }

    /**
     * Add event triggers and actions
     *
     * @param WebhookBuilderEvent $event
     */
    public function onWebhookBuild(WebhookBuilderEvent $event)
    {
        // add checkbox to the webhook form for new leads
        $type  = array(
            'label'       => 'caymland.bundlename.webhook.event.type.new',
            'description' => 'caymland.bundlename.webhook.event.type.new_desc', // note: we don't currently use a description, but may in the future so we have supported it here
        );

        // add it to the list
        $event->addEvent(YourBundle::ACTION_TO_TRIGGER, $type);

         // Note: you may create multiple arrays and call the $event->addEvent method multiple times
         // in this function to add several types all at once.
    }
}
?>
```

Add an event listener for the <code>WebhookEvents::WEBHOOK_ON_BUILD</code> event,
call the addEvent method and pass it an argument for your payload event constant,
and an array of a label and a description to be added to the Webhook user interface.

<code>YourBundle::ACTION_TO_TRIGGER</code> is a constant that
should be an event registered in your bundle.
We use the constant to save the type in the database and query for later.
You will use the same constant later, so its important to be consistent.

#### Payload Event Listener
<aside class="notice">
    Add an event listener which extends the <code>WebhookSubscriberBase</code> class. This event should be dispatched in your bundle when you wish to create a payload.
</aside>


```php
<?php

namespace Caymland\YourBundle\EventListener;

// its important to use the WebhookSubscriberBase event listener
// as it does a lot of heavy lifting for you.

use Caymland\WebhookBundle\EventListener\WebhookSubscriberBase;

// you should change this to your bundle's event class
use Caymland\LeadBundle\LeadEvents;

// you should change this to the event type that you are going to use.
// in our case it will be a lead event.
use Caymland\LeadBundle\Event\LeadEvent;

use JMS\Serializer\Serializer;
use Caymland\CoreBundle\Factory\CaymlandFactory;
use Doctrine\ORM\NoResultException;
use Caymland\CoreBundle\EventListener\CommonSubscriber;
use Caymland\LeadBundle\Event\PointsChangeEvent;
use Caymland\ApiBundle\Serializer\Exclusion\PublishDetailsExclusionStrategy;

/**
 * Class LeadSubscriber
 *
 * @package Caymland\Webhook\EventListener
 */
class LeadSubscriber extends WebhookSubscriberBase
{
    /**
     * @return array
     */
    static public function getSubscribedEvents ()
    {
        return array(
            // note this is the same constant we would have used in the addEvent() listener!
            LeadEvents::LEAD_POST_SAVE => array('onLeadNewUpdate', 0)
        )
    }

    /*
     * Generic method to execute when a lead does something
     */
    public function onLeadNewUpdate(LeadEvent $event)
    {
        // serializer groups are defined and optionally used in your bundle's entity.
        // Use these format your payload in JSON formats.
        $serializerGroups = array("leadDetails", "userList", "publishDetails", "ipAddress");

        // the beginning part of building our webhook payload of a lead entity
        $entity = $event->getLead();

        // our payload goes into an array
        $payload = array(
            'lead'      => $entity,
        );

        // we want to only trigger a payload if the lead is new
        if ($event->isNew()) {

            // again please note the use of the constant here - this should be consistent with the constant we register to the webhook form
            $webhookEvents = $this->getEventWebooksByType(LeadEvents::LEAD_POST_SAVE);

            // The webhook model is made available because we've extended the WebhookSubscriberBase class.
            $this->webhookModel->QueueWebhooks($webhookEvents, $payload, $serializerGroups);


        }
    }
}
```

The next step is to add an event listener for the action you want to actually use for creating payloads. This could be anything you want, your bundle will have to have an event type and use the event dispatcher to execute the event.

Here is an example listener to create a lead payload whenever a new lead is added to Caymland.

You can add the listener to any bundle. Be sure to register it in your bundle's config.php file.

<aside class="notice">
    The event constant that you return in the <code>getSubscriberEvents()</code> method should match the event type from the previous listener.
    This is used in a database query, so it must match exactly for the payload to be included in the hook POST to the endpoint URL.
</aside>

You can refer to the model for more documentation on the <code>QueueWebhooks</code> method.
In short you want to pass the event entities that this payload is being queued against
The payload, which is an array, and finally the serializer groups for formatting the JSON.

This should complete the set up, registering and executing custom webhook events and payloads.

#### Receiving Webhook Payloads

> A sample new lead post payload

```json
{"caymland.lead_post_save":{"lead":{"id":null,"points":0,"color":null,"fields":{"core":{"title":{"id":1,"group":"core","label":"Title","alias":"title","type":"lookup","value":null},"firstname":{"id":2,"group":"core","label":"First Name","alias":"firstname","type":"text","value":"Hello"},"lastname":{"id":3,"group":"core","label":"Last Name","alias":"lastname","type":"text","value":"World"},"company":{"id":4,"group":"core","label":"Company","alias":"company","type":"lookup","value":null},"position":{"id":5,"group":"core","label":"Position","alias":"position","type":"text","value":null},"email":{"id":6,"group":"core","label":"Email","alias":"email","type":"email","value":"example@email.com"},"phone":{"id":7,"group":"core","label":"Phone","alias":"phone","type":"tel","value":null},"mobile":{"id":8,"group":"core","label":"Mobile","alias":"mobile","type":"tel","value":null},"fax":{"id":9,"group":"core","label":"Fax","alias":"fax","type":"text","value":null},"address1":{"id":10,"group":"core","label":"Address Line 1","alias":"address1","type":"text","value":null},"address2":{"id":11,"group":"core","label":"Address Line 2","alias":"address2","type":"text","value":null},"city":{"id":12,"group":"core","label":"City","alias":"city","type":"lookup","value":null},"state":{"id":13,"group":"core","label":"State","alias":"state","type":"region","value":null},"zipcode":{"id":14,"group":"core","label":"Zipcode","alias":"zipcode","type":"lookup","value":null},"country":{"id":15,"group":"core","label":"Country","alias":"country","type":"country","value":null},"website":{"id":16,"group":"core","label":"Website","alias":"website","type":"text","value":null}},"social":{"twitter":{"id":17,"group":"social","label":"Twitter","alias":"twitter","type":"text","value":null},"facebook":{"id":18,"group":"social","label":"Facebook","alias":"facebook","type":"text","value":null},"googleplus":{"id":19,"group":"social","label":"Google+","alias":"googleplus","type":"text","value":null},"skype":{"id":20,"group":"social","label":"Skype","alias":"skype","type":"text","value":null},"instagram":{"id":21,"group":"social","label":"Instagram","alias":"instagram","type":"text","value":null},"foursquare":{"id":22,"group":"social","label":"Foursquare","alias":"foursquare","type":"text","value":null}},"personal":[],"professional":[]},"lastActive":null,"owner":null,"ipAddresses":[],"dateIdentified":null,"preferredProfileImage":null},"timestamp":"2015-08-18T18:53:33+00:00"}}
```

<aside class="notice">
 Click the <code>JSON</code> tab to see a sample lead JSON payload</code>.
</aside>


Webhooks enable Caymland to send data for leads, points, and email opens to outside applications. It does this by taking an outside application's endpoint url, and sending an HTTP post request to that location. In that post we include the relevant data the the event that has been fired.

To listen to the webhook posts, developers should create a publicly accessible endpoint location in their application. That endpoint should be available to receive a POST request from Caymland. The contents of the payload will vary based on the events that the user wishes to include in the payload.

An excellent way of testing to see the data that will be included is using [Requestb.in](http://requestb.in). Requestb.in allows users to view the full contents of the hook payload. Developers can refer to a post to the bin to see the contents and craft the way their application receives that input accordingly.
