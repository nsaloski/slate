### Extending Landing Pages

```php
<?php
// plugins/HelloWorldBundle/EventListener/PageSubscriber.php

namespace CaymlandPlugin\HelloWorldBundle\EventListener;

use Caymland\CoreBundle\EventListener\CommonSubscriber;
use Caymland\PageBundle\PageEvents;
use Caymland\PageBundle\Event\PageBuilderEvent;
use Caymland\PageBundle\Event\PageSendEvent;

/**
 * Class PageSubscriber
 */
class PageSubscriber extends CommonSubscriber
{

    /**
     * @return array
     */
    static public function getSubscribedEvents()
    {
        return array(
            PageEvents::PAGE_ON_BUILD   => array('onPageBuild', 0),
            PageEvents::PAGE_ON_DISPLAY => array('onPageDisplay', 0)
        );
    }

    /**
     * Register the tokens and a custom A/B test winner
     *
     * @param PageBuilderEvent $event
     */
    public function onPageBuild(PageBuilderEvent $event)
    {
        // Add page tokens
        $content = $this->templating->render('HelloWorldBundle:SubscribedEvents\PageToken:token.html.php');
        $event->addTokenSection('helloworld.token', 'plugin.helloworld.header', $content);

        // Add AB Test Winner Criteria
        $event->addAbTestWinnerCriteria(
            'helloworld.planetvisits',
            array(
                // Label to group by
                'group'    => 'plugin.helloworld.header',
                
                // Label for this specific a/b test winning criteria
                'label'    => 'plugin.helloworld.pagetokens.',
                
                // Static callback function that will be used to determine the winner
                'callback' => '\CaymlandPlugin\HelloWorldBundle\Helper\AbTestHelper::determinePlanetVisitWinner'
            )
        );
    }

    /**
     * Search and replace tokens with content
     *
     * @param PageSendEvent $event
     */
    public function onPageDisplay(PageSendEvent $event)
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

There are two way to extend pages: page tokens used to insert dynamic content into a page and a/b test winning criteria . Both leverage the `\Caymland\PageBundle\PageEvents::PAGE_ON_BUILD` event. Read more about [listeners and subscribers](#events).

#### Page Tokens

Page tokens are handled exactly the same as [Email Tokens](#page-tokens).

#### Page A/B Test Winner Criteria

Custom landing page A/B test winner criteria is handled exactly the same as [page A/B test winner criteria](#page-a/b-test-winner-criteria) with the only differences being that the `callback` function is passed `Caymland\PageBundle\Entity\Page $page` and `Caymland\PageBundle\Entity\Page $parent` instead. Of course `$children` is an ArrayCollection of Page entities as well.
