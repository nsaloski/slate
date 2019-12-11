### Extending Maintenance Cleanup

```php
<?php
// plugins\HelloWorldBundle\EventListener\MaintenanceSubscriber

namespace CaymlandPlugin\HelloWorldBundle\EventListener;

use Doctrine\DBAL\Connection;
use Caymland\CoreBundle\CoreEvents;
use Caymland\CoreBundle\Event\MaintenanceEvent;
use Caymland\CoreBundle\Factory\CaymlandFactory;

/**
 * Class MaintenanceSubscriber
 */
class MaintenanceSubscriber extends CommonSubscriber
{
    /**
     * @var Connection
     */
    protected $db;

    /**
     * MaintenanceSubscriber constructor.
     *
     * @param CaymlandFactory $factory
     * @param Connection    $db
     */
    public function __construct(CaymlandFactory $factory, Connection $db)
    {
        parent::__construct($factory);

        $this->db = $db;
    }

    /**
     * {@inheritdoc}
     */
    public static function getSubscribedEvents()
    {
        return [
            CoreEvents::MAINTENANCE_CLEANUP_DATA => ['onDataCleanup', -50]
        ];
    }

    /**
     * @param $isDryRun
     * @param $date
     *
     * @return int
     */
    public function onDataCleanup (MaintenanceEvent $event)
    {
        $qb = $this->db->createQueryBuilder()
            ->setParameter('date', $event->getDate()->format('Y-m-d H:i:s'));

        if ($event->isDryRun()) {
            $rows = (int) $qb->select('count(*) as records')
                ->from(CAYMLAND_TABLE_PREFIX.'worlds', 'w')
                ->where(
                    $qb->expr()->gte('w.date_added', ':date')
                )
                ->execute()
                ->fetchColumn();
        } else {
            $rows = (int) $qb->delete(CAYMLAND_TABLE_PREFIX.'worlds')
                ->where(
                    $qb->expr()->lte('date_added', ':date')
                )
                ->execute();
        }

        $event->setStat($this->translator->trans('caymland.maintenance.hello_world'), $rows, $qb->getSQL(), $qb->getParameters());
    }
}
```

To hook into the `caymland:maintenance:cleanup` command, create a listening for the `\Caymland\CoreBundle\CoreEvents::MAINTENANCE_CLEANUP_DATA` event. The event listener should check if data should be deleted or a counted. Use `$event->setStat($key, $affectedRows, $sql, $sqlParameters)` to give feedback to the CLI command. Note that `$sql` and `$sqlParameters` are only used for debugging and shown only in the dev environment.

<aside class="notice">
The plugin should check if a dry run is requested using `$event->isDryRun()` before deleting records!
</aside>
