### Extending Reports

```php
<?php
// plugins\HelloWorldBundle\EventListener\ReportSubscriber

namespace CaymlandPlugin\HelloWorldBundle\EventListener;

use Caymland\CoreBundle\EventListener\CommonSubscriber;
use Caymland\CoreBundle\Helper\GraphHelper;
use Caymland\ReportBundle\Event\ReportBuilderEvent;
use Caymland\ReportBundle\Event\ReportGeneratorEvent;
use Caymland\ReportBundle\Event\ReportGraphEvent;
use Caymland\ReportBundle\ReportEvents;
use Caymland\CoreBundle\Helper\Chart\ChartQuery;
use Caymland\CoreBundle\Helper\Chart\LineChart;

/**
 * Class ReportSubscriber
 */
class ReportSubscriber extends CommonSubscriber
{

    /**
     * @return array
     */
    static public function getSubscribedEvents ()
    {
        return array(
            ReportEvents::REPORT_ON_BUILD          => array('onReportBuilder', 0),
            ReportEvents::REPORT_ON_GENERATE       => array('onReportGenerate', 0),
            ReportEvents::REPORT_ON_GRAPH_GENERATE => array('onReportGraphGenerate', 0)
        );
    }

    /**
     * Add available tables, columns, and graphs to the report builder lookup
     *
     * @param ReportBuilderEvent $event
     *
     * @return void
     */
    public function onReportBuilder (ReportBuilderEvent $event)
    {
        // Use checkContext() to determine if the report being requested is this report
        if ($event->checkContext(array('worlds'))) {
            // Define the columns that are available to the report.
            $prefix  = 'w.';
            $columns = array(
                $prefix . 'visit_count' => array(
                    'label' => 'caymland.hellobundle.report.visit_count',
                    'type'  => 'int'
                ),
                $prefix . 'world' => array(
                    'label' => 'caymland.hellobundle.report.world',
                    'type'  => 'text'
                ),
            );

             // Several helper functions are available to append common columns such as categories, publish state fields, lead, etc.  Refer to the ReportBuilderEvent class for more details.
            $columns = $filters = array_merge($columns, $event->getStandardColumns($prefix), $event->getCategoryColumns());

            // Optional to override and update filters, i.e. change it to a select list for the UI
            $filters[$prefix.'world']['type'] = 'select';
            $filters[$prefix.'world']['list'] = array(
                'earth' => 'Earth',
                'mars'  => 'Mars'
            );

            // Add the table to the list
            $event->addTable('worlds',
                array(
                    'display_name' => 'caymland.helloworld.worlds',
                    'columns'      => $columns,
                    'filters'      => $filters // Defaults to columns if not set
                )
            );

            // Register available graphs; can use line, pie, or table
            $event->addGraph('worlds', 'line', 'caymland.hellobundle.graph.line.visits');
        }
    }

    /**
     * Initialize the QueryBuilder object used to generate the report's data.
     * This should use Doctrine's DBAL layer, not the ORM so be sure to use
     * the real schema column names (not the ORM property names) and the
     * CAYMLAND_TABLE_PREFIX constant.
     *
     * @param ReportGeneratorEvent $event
     *
     * @return void
     */
    public function onReportGenerate (ReportGeneratorEvent $event)
    {
        $context = $event->getContext();
        if ($context == 'worlds') {
            $qb = $event->getQueryBuilder();

            $qb->from(CAYMLAND_TABLE_PREFIX . 'worlds', 'w');
            $event->addCategoryLeftJoin($qb, 'w');

            $event->setQueryBuilder($qb);
        }
    }

    /**
     * Generate the graphs
     *
     * @param ReportGraphEvent $event
     *
     * @return void
     */
    public function onReportGraphGenerate (ReportGraphEvent $event)
    {
        if (!$event->checkContext('worlds')) {
            return;
        }

        $graphs   = $event->getRequestedGraphs();
        $qb       = $event->getQueryBuilder();

        foreach ($graphs as $graph) {
            $queryBuilder = clone $qb;
            $options      = $event->getOptions($graph);
            /** @var ChartQuery $chartQuery */
            $chartQuery    = clone $options['chartQuery'];
            $chartQuery->applyDateFilters($queryBuilder, 'date_added', 'v');

            switch ($graph) {
                case 'caymland.hellobundle.graph.line.visits':
                    $chart = new LineChart(null, $options['dateFrom'], $options['dateTo']);
                    $chartQuery->modifyTimeDataQuery($queryBuilder, 'date_added', 'v');
                    $visits = $chartQuery->loadAndBuildTimeData($queryBuilder);
                    $chart->setDataset($options['translator']->trans('caymland.hellobundle.graph.line.visits'), $visits);
                    $data         = $chart->render();
                    $data['name'] = $graph;
                    $data['iconClass'] = 'fa-tachometer';
                    $event->setGraph($graph, $data);

                    break;
            }
        }
    }
}
```

Adding and rendering custom reports are done by listening to the `\Caymland\ReportBundle\ReportEvents::REPORT_ON_BUILD`, `ReportEvents::REPORT_ON_GENERATE` and `ReportEvents::REPORT_ON_GRAPH_GENERATE` events.

#### Defining the Report

Defining the report is done through the `ReportEvents::REPORT_ON_BUILD` event. This is where the plugin will define the context of the report, available columns for table data, available filters for the table data (defaults to columns) and available graphs. See the code example's `onReportBuilder` for details.

##### Column Definition

Each column array can include the following properties:

Key|Required|Type|Description
---|--------|----|-----------
**label**|REQUIRED|string|The language string for the column
**type**|REQUIRED|string|Column type
**alias**|OPTIONAL|string|An alias for the returned value. Useful in conjuction with `formula`
**formula**|OPTIONAL|string|SQL formula instead of a column. e.g. `SUBSTRING_INDEX(e.type, \'.\', 1)`
**link**|OPTIONAL|string|Route name to convert the value into a hyperlink. Used usually with an ID of an Entity. The route must accept `objectAction` and `objectId` parameters.

##### Filter Definition

Filter definitions are optional as Caymland will default to the column list. But sometimes it's useful to replace filter values with a select list. Filter definitions can accept the same properties as columns but can also accept the following:

Key|Required|Type|Description
---|--------|----|-----------
**list**|OPTIONAL|array|Used when `type` is `select` for a filter. Provides the dropdown options for a select input. Format should be `value => label`
**operators**|OPTIONAL|array|Custom list of operators to allow for this filter. See `Caymland\ReportBundle\Builder\CaymlandReportBuilder::OPERATORS` for a examples.

#### Generate the QueryBuilder

The `ReportEvents::REPORT_ON_GENERATE` event is dispatched when a report is to be generated and displayed. In this function, the plugin should define the QueryBuilder object used to generate the table data. 

Use `$event->checkContext()` to determine if the report requested is the subscribers report.

<aside class="notice">
Note that the <code>ReportEvents::REPORT_ON_GENERATE</code> event should use Doctrine's DBAL layer QueryBuilder obtained via <code>$qb = $event->getQueryBuilder();</code>.
</aside>

There are a number of helper functions to append joins for commonly used relationships such as category, leads, ip address, etc.  Refer to the ReportGeneratorEvent class for more details.
   
#### Graphs

Graphs are generated using `ReportEvents::REPORT_ON_GRAPH_GENERATE` event. The listener should check the context then generate and set the graph data. There are several classes to assist with generating classes. See 
