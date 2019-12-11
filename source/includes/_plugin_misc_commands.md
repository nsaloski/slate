### Commands

Support for new CLI commands can be added using [Symfony's console component](http://symfony.com/doc/2.8/console.html).  

#### Moderated Commands

```php
<?php
// plugins\HelloWorldBundle\Command\WorldCommand.php

namespace CaymlandPlugin\HelloWorldBundle\Command;

use Caymland\CoreBundle\Command\ModeratedCommand;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

/**
 * CLI Command to send a scheduled broadcast.
 */
class WorldCommand extends ModeratedCommand
{
    /**
     * {@inheritdoc}
     */
    protected function configure()
    {
        // ...

        // Important to append options used by ModeratedCommand
        parent::configure();
    }

    /**
     * @param InputInterface  $input
     * @param OutputInterface $output
     *
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // Validate if the command is already has an instance running
        // A third argument can be passed here if this command is running something unique such as an ID
        if (!$this->checkRunStatus($input, $output)) {
            return 0;
        }

        // Execute some stuff
        
        // Complete this execution
        $this->completeRun();

        return 0;
    }
}
```

Caymland provide an method for moderating commands meaning it will only allow one instance to run at a time. To utilize this method, extend the `Caymland\CoreBundle\Command\ModeratedCommand` class.

