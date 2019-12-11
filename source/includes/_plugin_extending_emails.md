### Extending Emails

```php
<?php
// plugins/HelloWorldBundle/EventListener/EmailSubscriber.php

namespace CaymlandPlugin\HelloWorldBundle\EventListener;

use Caymland\CoreBundle\EventListener\CommonSubscriber;
use Caymland\EmailBundle\EmailEvents;
use Caymland\EmailBundle\Event\EmailBuilderEvent;
use Caymland\EmailBundle\Event\EmailSendEvent;

/**
 * Class EmailSubscriber
 */
class EmailSubscriber extends CommonSubscriber
{

    /**
     * @return array
     */
    static public function getSubscribedEvents()
    {
        return array(
            EmailEvents::EMAIL_ON_BUILD   => array('onEmailBuild', 0),
            EmailEvents::EMAIL_ON_SEND    => array('onEmailGenerate', 0),
            EmailEvents::EMAIL_ON_DISPLAY => array('onEmailGenerate', 0)
        );
    }

    /**
     * Register the tokens and a custom A/B test winner
     *
     * @param EmailBuilderEvent $event
     */
    public function onEmailBuild(EmailBuilderEvent $event)
    {
        // Add email tokens
        $content = $this->templating->render('HelloWorldBundle:SubscribedEvents\EmailToken:token.html.php');
        $event->addTokenSection('helloworld.token', 'plugin.helloworld.header', $content);

        // Add AB Test Winner Criteria
        $event->addAbTestWinnerCriteria(
            'helloworld.planetvisits',
            array(
                // Label to group by
                'group'    => 'plugin.helloworld.header',
                
                // Label for this specific a/b test winning criteria
                'label'    => 'plugin.helloworld.emailtokens.',
                
                // Static callback function that will be used to determine the winner
                'callback' => '\CaymlandPlugin\HelloWorldBundle\Helper\AbTestHelper::determinePlanetVisitWinner'
            )
        );
    }

    /**
     * Search and replace tokens with content
     *
     * @param EmailSendEvent $event
     */
    public function onEmailGenerate(EmailSendEvent $event)
    {
        // Get content
        $content = $event->getContent();

        // Search and replace tokens
        $content = str_replace('{hello}', 'world!', $content);

        // Set updated content
        $event->setContent($content);
    }
}
```
There are two way to extend emails: email tokens used to insert dynamic content into an email and a/b test winning criteria . Both leverage the `\Caymland\EmailBundle\EmailEvents::EMAIL_ON_BUILD` event. Read more about [listeners and subscribers](#events).

#### Email Tokens

Email tokens are placeholders that are inserted into an email that can be replaced by dynamic content when the email is sent or viewed in the browser.

`$event->addTokenSection($uniqueId, $headerTranslationKey, $htmlContent)` is used to generate the section for drag and drop tokens in the email builder.  

`$uniqueId` must be unique. 

`$headerTranslationKey` is the translation key that will be used to create the section's header.

`$htmlContent` is the HTML that will be inserted into the builder's token list for this token's section. `$this->templating->render()` can be used to render a specific view's content (using [view notation](#views). There is free reign as to what the HTML will look like but the important part is that the elements representing the tokens must have the attribute `data-token="{token_text}"` in order for the builder to recognize them.
 
For example, `<a href="#" data-token="{hello}" class="btn btn-default btn-block">Translated Token Text</a>`

##### Custom Token Handling
 
To convert the token into a link while requesting what the links text should be, use the attributes `data-token='<a href="%url={hello}%">%text%</a>' data-drop="showBuilderLinkModal"` (replacing `hello` with the plugin's custom token).

To request simple feedback from the user and inject it into the token, use the attributes `data-token='{hello=%world%}' data-drop="showBuilderFeedbackModal"`. A modal will appear with a simple input box. Whatever the user inputs will replace `%world%`.
  
If you need more control or more customization, create and define a custom JS function within the Caymland namespace (i.e. `Caymland.customFunction` and use the attribute `data-drop="customFunction"`. When a user drops the token, `Caymland.customFunction()` passing the arguments event (jQuery Event), ui (jQuery UI object with draggable, helper, etc), editorId (ID of the inline CkEditor).

<pre class="inline">
Caymland.customFunction = function (event, ui, editorId) {
    var token  = mQuery(ui.draggable).data('token');
    
    // Do something fancy then insert token
        
    Caymland.insertBuilderEditorToken(editorId, token);
};
</pre>

#### Email A/B Test Winner Criteria

To add a custom an a/b test winner criteria, i.e. a test to compare specific aspects of an email to determine what made it more successful over it's variants, use `$event->addAbTestWinnerCriteria($uniqueId, $parameters)`.
 
`$parameters` can have the following elements:
 
Key|Required|Type|Description
---|--------|----|-----------
**group**|REQUIRED|string|Translation string to group criteria by in the dropdown select list
**label**|OPTIONAL|string|Label for this option in the drop down list
**callback**|REQUIRED|mixed|Static callback function that will be called to determine the winner when the email detail page is viewed
**formType**|OPTIONAL|string|The alias of a custom form type used to set config options for the decision
**formTypeOptions**|OPTIONAL|array|Array of options to include into the formType's $options argument
**formTypeCleanMasks**|OPTIONAL|array|Array of input masks to clean a values from formType
**formTypeTheme**|OPTIONAL|string|Theme to customize elements for formType
The callback can accept the following variables (determined via ReflectionMethod::invokeArgs()):

Variable|Type|Description
--------|----|-----------
**$properties**|array|Array of elements saved from the configured formType; keyed by email ID in the case of multiple variants
**$factory**|Caymland\CoreBundle\Factory\CaymlandFactory|[Caymland's factory service](#factory-service)
**$email**|Caymland\EmailBundle\Entity\Email|Email entity for the displayed email
**$parent**|Caymland\EmailBundle\Entity\Email|Email entity for the parent of the email entity
**$children**|Doctrine\Common\Collections\ArrayCollection|All variants of the parent email

The callback function should return an array keyed with the following elements:

Key|Type|Description
---|----|-----------
**winners**|array|Array of IDs of the winners (array in case of a tie)
**support**|mixed|Passed to the view defined by supportTemplate below in order to render visual support for the winners (such as a graph, etc)
**supportTemplate**|string|View notation to render content for the A/B stats modal. For example, `HelloWorldBundle:SubscribedEvents\AbTest:graph.html.php`

<pre class="inline">
return array(
   'winners'         => $winners,
   'support'         => $support,
   'supportTemplate' => 'HelloWorldBundle:SubscribedEvents\AbTest:graph.html.php'
);
</pre>

#### Monitored Inbox Integration
```php
<?php
// plugins/HelloWorldBundle/EventListener/MonitoredInboxSubscriber.php

namespace CaymlandPlugin\HelloWorldBundle\EventListener;

use Caymland\CoreBundle\EventListener\CommonSubscriber;
use Caymland\EmailBundle\EmailEvents;
use Caymland\EmailBundle\Event\MonitoredEmailEvent;
use Caymland\EmailBundle\Event\ParseEmailEvent;
use Caymland\EmailBundle\MonitoredEmail\Mailbox;

/**
 * Class MonitoredInboxSubscriber
 */
class MonitoredInboxSubscriber extends CommonSubscriber
{
    private $bundle = 'HelloWorldBundle';
    private $monitor =  'deep_space_emails';

    /**
     * @return array
     */
    static public function getSubscribedEvents()
    {
        return [
            EmailEvents::MONITORED_EMAIL_CONFIG => ['onConfig', 0],
            EmailEvents::EMAIL_PRE_FETCH        => ['onPreFetch', 0],
            EmailEvents::EMAIL_PARSE            => ['onParse', 0],
        ];
    }

    /**
     * Inject the IMAP folder settings into the Configuration
     *
     * @param MonitoredEmailEvent $event
     */
    public function onConfig(MonitoredEmailEvent $event)
    {
        /**
         * The first argument is something unique to recognize this plugin.
         * The second argument should be something unique to identify this monitored inbox.
         * The third argument is the label for this monitored inbox.
         */
         $event->addFolder($this->bundle, $this->monitor, 'caymland.world.monitored_deep_space_emails');
    }

    /**
     * Inject search criteria for which messages to fetch from the configured folder.
     *
     * @param ParseEmailEvent $event
     */
    public function onPreFetch(ParseEmailEvent $event)
    {
        $event->setCriteriaRequest($this->bundle, $this->monitor, Mailbox::CRITERIA_UNSEEN. " " . Mailbox::CRITERIA_FROM ." aliens@andromeda");
    }

    /**
     * Parse the messages
     *
     * @param ParseEmailEvent $event
     */
    public function onParse(ParseEmailEvent $event)
    {
        if ($event->isApplicable($this->bundle, $this->monitor)) {
            $messages = $event->getMessages();

            /** @var \Caymland\EmailBundle\MonitoredEmail\Message $message */
            foreach ($messages as $message) {
                // Do something
            }
        }
    }
}
```

Plugins have access to hook into the `caymland:email:fetch` command to fetch email from a specific inbox/folder and process the content of the message. Starting in 2.1.1, the plugin also has access to inject specific search criteria for the messages to be processed.

To do this, the plugin needs to add an event listener for three events:

1. `EmailEvents::MONITORED_EMAIL_CONFIG` This event is dispatched to inject the fields into Caymland's Configuration to configure the IMAP inbox and folder that should be monitored.
2. `EmailEvents::EMAIL_PRE_FETCH` This event is dispatched during the execution of the `caymland:email:fetch` command. It's used to inject search criteria for the messages desired.
3. `EmailEvents::EMAIL_PARSE` This event parses the messages fetched by the command.
