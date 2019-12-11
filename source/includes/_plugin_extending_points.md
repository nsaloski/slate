### Extending Points

Custom point actions and triggers can be added by listening to their respective on build events. Read more about [listeners and subscribers](#events).

#### Point Actions

To add a custom point action used to give a lead x points for doing a certain action, add a listener to the `\Caymland\PointBundle\PointEvents::POINT_ON_BUILD` event then configure the custom point action with `$event->addAction($identifier, $parameters)` method. `$identifier` must be something unique. The `$parameters` array can contain the following elements:


Key|Required|Type|Description
---|--------|----|-----------
**label**|REQUIRED|string|The language string for the option in the dropdown
**formType**|OPTIONAL|string|The alias of a custom form type used to set config options. 
**formTypeOptions**|OPTIONAL|array|Array of options to include into the formType's $options argument
**formTypeCleanMasks**|OPTIONAL|array|Array of input masks to clean a values from formType
**formTypeTheme**|OPTIONAL|string|Theme to customize elements for formType     
**template**|OPTIONAL|string|View template used to render the formType
**callback**|OPTIONAL|mixed|Static callback function used to validate the action. Return true to add the points to the lead.

In order for the custom point action to work, add something like the following in the code logic when the lead executes the custom action:
 
 
`$this->getModel('point')->triggerAction('page.hit', $event->getHit());`


#### Point Triggers


To add a custom point trigger used to execute a specific action once a lead hits X number of points, add a listener to the `\Caymland\PointBundle\PointEvents::TRIGGER_ON_BUILD` event then configure the custom point trigger with `$event->addEvent($identifier, $parameters)` method. `$identifier` must be something unique. The `$parameters` array can contain the following elements:

Key|Required|Type|Description
---|--------|----|-----------
**label**|REQUIRED|string|The language string for the option in the dropdown
**formType**|OPTIONAL|string|The alias of a custom form type used to set config options. 
**formTypeOptions**|OPTIONAL|array|Array of options to include into the formType's $options argument
**formTypeCleanMasks**|OPTIONAL|array|Array of input masks to clean a values from formType
**formTypeTheme**|OPTIONAL|string|Theme to customize elements for formType     
**template**|OPTIONAL|string|View template used to render the formType
**callback**|OPTIONAL|mixed|Static callback function used to execute the custom action.
