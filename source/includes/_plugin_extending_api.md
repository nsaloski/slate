### Extending API
```php
<?php
// plugins/HelloWorldBundle/Config/config.php

return array(
    // ...

    'routes'   => array(
        
        // ...
        
        'api' => array(
            'plugin_helloworld_api' => array(
                'path'       => '/hello/worlds',
                'controller' => 'HelloWorldBundle:Api:worlds',
                'method'     => 'GET'
            )
        )
    ),

    // ...
);
```

```php
<?php
// plugins/HelloWorldBundle/Controller/ApiController.php

namespace Caymland\LeadBundle\Controller\Api;

use FOS\RestBundle\Util\Codes;
use Caymland\ApiBundle\Controller\CommonApiController;

class ApiController extends CommonApiController
{

    /**
     * Get a list of worlds
     * 
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function getWorldsAction()
    {
        if (!$this->get('caymland.security')->isGranted('plugin:helloWorld:worlds:view')) {
            return $this->accessDenied();
        }

        $filter  = $this->request->query->get('filter', null);
        $limit   = $this->request->query->get('limit', null);
        $start   = $this->request->query->get('start', null);
        
        /** @var \CaymlandPlugin\HelloWorldBundle\Model\WorldsModel $model */
        $model   = $this->getModel('helloworld.worlds');

        $worlds  = $model->getWorlds($filter, $limit, $start);
        $worlds  = $this->view($worlds, Codes::HTTP_OK);
        
        return $this->handleView($worlds);
    }
}
```

To add custom API endpoints, simply define the routes under the API firewall in the [plugin's config file](#routes). This will place the route behind /api which will only be accessible if the requester has been authorized via OAuth.
 
The api controller(s), should extend `Caymland\ApiBundle\Controller\CommonApiController` to leverage the helper methods provided and to utilize the REST views.
