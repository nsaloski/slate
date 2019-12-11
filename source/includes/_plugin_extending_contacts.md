### Extending Contacts (Leads)

#### Contact Timeline/History
```php
<?php
// plugins/HelloWorldBundle/EventListener/LeadSubscriber.php

namespace CaymlandPlugin\HelloWorldBundle\EventListener;

use Caymland\CoreBundle\EventListener\CommonSubscriber;
use Caymland\LeadBundle\Event\LeadTimelineEvent;
use Caymland\LeadBundle\LeadEvents;

/**
 * Class LeadSubscriber
 */
class LeadSubscriber extends CommonSubscriber
{

    /**
     * @return array
     */
    static public function getSubscribedEvents()
    {
        return [
            LeadEvents::TIMELINE_ON_GENERATE => ['onTimelineGenerate', 0]
        ];
    }

    /**
     * Compile events for the lead timeline
     *
     * @param LeadTimelineEvent $event
     */
    public function onTimelineGenerate(LeadTimelineEvent $event)
    {
        // Add this event to the list of available events which generates the event type filters
        $eventTypeKey  = 'visited.worlds';
        $eventTypeName = $this->translator->trans('caymland.hello.world.visited_worlds');
        $event->addEventType($eventTypeKey, $eventTypeName);

        // Determine if this event has been filtered out
        if (!$event->isApplicable($eventTypeKey)) {

            return;
        }

        /** @var \CaymlandPlugin\HelloWorldRepository\Entity\WorldRepository $repository */
        $repository = $this->em->getRepository('HelloWorldBundle:World');

        // $event->getQueryOptions() provide timeline filters, etc.
        // This method should use DBAL to obtain the events to be injected into the timeline based on pagination
        // but also should query for a total number of events and return an array of ['total' => $x, 'results' => []].
        // There is a TimelineTrait to assist with this. See repository example.
        $stats = $repository->getVisitedWorldStats($event->getLead()->getId(), $event->getQueryOptions());

        // If isEngagementCount(), this event should only inject $stats into addToCounter() to append to data to generate
        // the engagements graph. Not all events are engagements if they are just informational so it could be that this
        // line should only be used when `!$event->isEngagementCount()`. Using TimelineTrait will determine the appropriate
        // return value based on the data included in getQueryOptions() if used in the stats method above.
        $event->addToCounter($eventTypeKey, $stats);

        if (!$event->isEngagementCount()) {
            // Add the events to the event array
            foreach ($stats['results'] as $stat) {
                if ($stat['dateSent']) {
                    $event->addEvent(
                        [
                            // Event key type
                            'event'           => $eventTypeKey,
                            // Event name/label - can be a string or an array as below to convert to a link
                            'eventLabel'      => [
                                'label' => $stat['name'],
                                'href'  => $this->router->generate(
                                    'caymland_dynamicContent_action',
                                    ['objectId' => $stat['dynamic_content_id'], 'objectAction' => 'view']
                                )
                            ],
                            // Translated string displayed in the Event Type column
                            'eventType'       => $eventTypeName,
                            // \DateTime object for the timestamp column
                            'timestamp'       => $stat['dateSent'],
                            // Optional details passed through to the contentTemplate
                            'extra'           => [
                                'stat' => $stat,
                                'type' => 'sent'
                            ],
                            // Optional template to customize the details of the event in the timeline
                            'contentTemplate' => 'CaymlandDynamicContentBundle:SubscribedEvents\Timeline:index.html.php',
                            // Font Awesome class to display as the icon
                            'icon'            => 'fa-envelope'
                        ]
                    );
                }
            }
        }
    }
}
```

##### TIMELINE_ON_GENERATE Event Listener

To inject events into a contact's timeline, create an event listener that listens to the `LeadEvents::TIMELINE_ON_GENERATE` event. Using this event, the plugin can inject unique items into the timeline and also into the engagements graph on each page. 

The event listener will receive a `Caymland\LeadBundle\Event\LeadTimelineEvent` object. The commonly used methods are defined below:

Method|Description
------|----------
**isApplicable**|Determines if this event is applicable and not filtered out.
**addEventType()**|Required - Add this event to the list of available events.
**getLead()**|Get the Lead entity the event is dispatched for
**getQueryOptions()**|Used to get pagination, filters, etc needed to generate an appropriate query 
**addToCounter()**|Used to add total number of events (across all pages) to the counters. This also generates the numbers for the engagements graph.
**addEvent()**|Required - Injects an event into the timeline. Accepts an array with the keys defined as below. 

**addEvent($event) Key Definitions**
Key|Required|Type|Description
---|--------|----|-----------
**event**|REQUIRED|string|The key for this event. Eg. world.visited
**eventType**|REQUIRED|string|The translated string representing this event type. Eg. Worlds visited
**timestamp**|REQUIRED|\DateTime|DateTime object when this event took place
**eventLabel**|OPTIONAL|string/array|The translated string to display in the event name. Examples include names of items, page titles, etc. This can also be an array of ['label' => '', 'href' => ''] to have the entry converted to a link. This will default to eventType if not defined.
**extra**|OPTIONAL|array|Whatever should be passed through to the content template to generate the details view for this event
**contentTemplate**|OPTIONAL|string|Template that should be used to generate the details view for this event. Eg. HelloBundle:SubscribedEvents\Timeline:index.html.php
**icon**|OPTIONAL|Font Awesome class

<div class="clear-right"></div>

##### TIMELINE_ON_GENERATE Repository Method

```php
<?php
//plugins/HelloWorldBundle/Entity/WorldRepository.php

namespace Caymland\LeadBundle\Entity;

use Caymland\CoreBundle\Entity\CommonRepository;
use Caymland\LeadBundle\Entity\TimelineTrait;

/**
 * Class WorldRepository
 */
class WorldRepository extends CommonRepository
{
    use TimelineTrait;

    /**
     * @param       $leadId
     * @param array $options
     */
    public function getTimelineStats($leadId, array $options = [])
    {
        $query = $this->getEntityManager()->getConnection()->createQueryBuilder();

        $query->select('w.id, w.name, w.visited_count, w.date_visited, w.visit_details')
            ->from(CAYMLAND_TABLE_PREFIX.'world_visits', 'w')
            ->where($query->expr()->eq('w.lead_id', (int) $leadId));

        if (isset($options['search']) && $options['search']) {
            $query->andWhere(
                $query->expr()->like('w.name', $query->expr()->literal('%' . $options['search'] . '%'))
            );
        }

        return $this->getTimelineResults($query, $options, 'w.name', 'w.date_visited', ['visit_details'], ['date_visited']);
    }
}
```

To assist with the generation of events, the `Caymland\LeadBundle\Entity\TimelineTrait` has been created.

To leverage this, accept the array from `$event->getQueryOptions()` in the repository method. Create a DBAL QueryBuilder object (`$this->getEntityManager()->getConnection()->createQueryBuilder()`) and define the basics of the array, including filtering by lead id and search filter. Then pass the QueryBuilder object to the `getTimelineResults()` method along with the following arguments:

Key|Required|Type|Description
---|--------|----|-----------
**$query**|REQUIRED|QueryBuilder|DBAL QueryBuilder object defining basics of the query.
**$options**|REQUIRED|array|Array generated and passed into method by $event->getQueryOptions() in the event listener above
**$eventNameColumn**|REQUIRED|string|Name of the column (with table prefix) that should be used when sorting by event name
**$timestampColumn**|REQUIRED|string|Name of the column (with table prefix) that should be used when sorting by timestamp
**$serializedColumns**|OPTIONAL|array|When using DBAL, arrays are not auto unserialized by Doctrine. Define the columns here (as returned by the query results) to auto unserialize.
**$dateTimeColumns**|OPTIONAL|array|When using DBAL, datetime columns are not auto converted to \DateTime objects by Doctrine. Define the columns here (as returned by the query results) to auto do so.     
**$resultsParserCallback**|OPTIONAL|callback|Callback to custom parse a result. This is optional and mainly used to handle a column result when all results are already being looped over for $serializedColumns and $dateTimeColumns.
