### Views
```php
<?php
//plugins/HelloWorldBundle/Views/World/details.html.php

// Check if the request is Ajax
if (!$app->getRequest()->isXmlHttpRequest()) {

    // Set tmpl for parent template
    $view['slots']->set('tmpl', 'Details');
    
    // Extend index.html.php as the parent
    $view->extend('HelloWorldBundle:World:index.html.php');
}
?>

<div>
    <!-- Desired content/markup -->
</div>
```

```php
<?php
// plugins/HelloWorldBundle/Views/World/index.html.php

// Extend the base content
$view->extend('CaymlandCoreBundle:Default:content.html.php');

// Get tmpl from sub-template
$tmpl = $view['slots']->get('tmpl', 'Details');

// Tell Caymland to call JS onLoad method
$view['slots']->set('caymlandContent', 'helloWorld'.$tmpl);

// Set the page and header title
$header = ($tmpl == 'World')
    ? $view['translator']->trans(
        'plugin.helloworld.worlds',
        array('%world%' => ucfirst($world))
    ) : $view['translator']->trans('plugin.helloworld.manage_worlds');
$view['slots']->set('headerTitle', $header);
?>

<div class="helloworld-content">
    <?php $view['slots']->output('_content'); ?>
</div>
```

Views take data given it from the controller and displays the content to the user. Templates can be called from the controller or from within other templates.

It was discussed in the Controller's [delegateView() method](#delegateredirect($url)) how to render a view from a controller. Remember that delegateView uses contentTemplate to determine what view to render.

Similar to controller notation, views are noted as `HelloWorldBundle:Contact:form.html.php` which will point to the file `/path/to/caymland/plugins/HelloWorldBundle/Views/Contact/form.html.php`. 

<aside class="notice">
View notation is in the format of <code>BundleName:ViewName:template.html.php</code>. To use a view that has been nested in Views, use <code>BundleName:ViewName\Subfolder:template.html.php</code>.
</aside>

#### View Parameters

Remember the array passed via viewParameters in the Controller's [delegateView() method](#delegateview($args))? Here is where the elements of that array will be made available as variables.  
 
 For example, `$world` will be available and assigned the value of `mars` from the following example: 
 
 <pre class="inline">
 'viewParameters'  => array(
     'world' => 'mars'
 ),
 </pre>
 
By default there are a couple variables available and should not be overridden by the controller

Variable|Description
----|----
**$view**|Contains all the helper objects available to templates along with providing methods for extending and/or rendering other templates.
**$app**|Gives access to request and session to views via `$app->getRequest()` and `$app->getSession()`

#### Extending Views
```php
<?php

// Extends the full document with menu, page header, etc
$view->extend('CaymlandCoreBundle:Default:content.html.php');
```

```php
<?php

// Extends the "slim" document which includes just the head and body with main content; does not include menu, page header, etc.
$view->extend('CaymlandCoreBundle:Default:content.html.php');
```

Because of the use of Ajax, views must also be able to return main content vs the entire document. This is done by extending Caymland's base templates. Notice the notation is the same as what is used in controllers.

To determine if the request is Ajax or not, use `$app->getRequest()->isXmlHttpRequest()`. Another option is to make the determination in the controller and pass it in the `viewParameters` array.

Of course, this can also be used to extend custom templates as demonstrated in [this code example](#views).

<aside>
The controller should call the sub-template.
</aside>

<aside class="notice">
It may help to note that extended templates are rendered inside out. So if <code>HelloWorldBundle:World:details.html.php</code> extends <code>HelloWorldBundle:World:index.html.php</code> which extends <code>CaymlandCoreBundle:Default:content.html.php</code>, the content of details.html.php will be rendered first and injected into index.html.php as it is rendered. <code>HelloWorldBundle:World:template1.html.php</code> will be rendered last.
</aside>

To include the content from the sub-template in the parent template, use `$view['slots']->output('_content');`. See the [Slots Helper](#slots-helper) for more information.

#### Rendering Views within Views

To render the content of another template from within a template, simply use `echo $view->render('BundleName:ViewName:template.html.php', array('parameter' => 'value'));`

#### Template Helpers

There are a number of template helper objects and helper view templates that are built into Caymland.

#####Slots Helper

```php
<?php

// Set a slot with content; will overwrite if slot already exists 
$view['slots']->set('name', 'the content');

// Append content to slot rather than overwrite
$view['slots']->append('name', ' and more content');
$view['slots']->append('existingArray', 
    array(
        'append' => 'me'
    )
);

// Get the slot content
$content = $view['slots']->get('name', 'default value');

// Output the content; note that echo is not required
$view['slots']->output('name');

// Check if a slot exists
if ($view['slots']->has('name')) {
    // Do something
}
```

As seen with extending views, the slots helper is used to pass content up through parent templates. Remember that extended templates are rendered inside out. So, content can be set in a sub template and accessed from the parent template but content set in a parent template, will not be available to the sub-template.

<div class="clear-right"></div>
#####Asset Helper
```php
<?php

// Generate relative URL to image
echo '<img src="' . $view['assets']->getUrl('plugins/HelloWorldBundle/assets/images/earth.png') . '" />';

// Dynamically insert script into head
echo $view['assets']->includeScript('plugins/HelloWorldBundle/assets/helloworld.js');

// Dynamically insert stylesheet into head
echo $view['assets']->includeStylesheet('plugins/HelloWorldBundle/assets/helloworld.css');
```
The asset helper, accessed via `$view['assets']`, is used to load assets into the DOM including images, script and stylesheets. 

<aside class="notice">
<code>$view['assets']</code> should always be used to ensure that assets work with Caymland installed in the web root, installed in a subdirectory, ran under the dev environment (index_dev.php), and/or ran under the prod environment.
</aside>

The asset helper also provides a way to insert scripts and stylesheets into the head for ajax loaded content using `$view['assets']->includeScript()` and `$view['assets']->includeStylesheet()`.

<div class="clear-right"></div>
#####Router Helper

```php
<a href="<?php echo $view['router']->generate('plugin_helloworld_world', array('world' => 'mars')); ?>" data-toggle="ajax" />Mars</a>
```

The router helper, `$view['router']`, works as explained in the [Router](#router).

<div class="clear-right"></div>
#####Translation Helper

```php
<h1><?php echo $view['translator']->trans('plugin.helloworld.worlds', array('%world%', 'Mars'); ?></h1>
```

The translation helper, `$view['translator']`, works as explained in the [Translator](#translator) section.

<div class="clear-right"></div>
#####Date Helper

```php
<?php

// Can be string or \DateTime object; if string, it's assumed local time unless noted otherwise via third argument to helper methods
$datetime = '2015-04-12 20:56:00';

// Formats per full date system config setting 
$fullDateTime = $view['date']->toFull($datetime);

// Formats per short date system config setting
$shortDateTime = $view['date']->toShort($datetime);

// Formats per date only system config setting
$date = $view['date']->toDate($datetime);

// Formats per time only system config setting
$time = $view['date']->toTime($datetime);

// Formats as date only config setting + time only config setting
$datetime = $view['date']->toFullConcat($datetime);

// Formats date as Yesterday, 8:02 pm (Today or Tomorrow); otherwise 'x days ago'  
$text = $view['date']->toText($datetime);

// Format a string that is not already in local time
$fullDateTime = $view['date']->toFull($datetime, 'Y-m-d H:i:s', 'UTC');
```
The date helper can be used to format dates based on system and/or user settings. 

The first argument to the various methods can either be a date/time string or a \DateTime object. If a string, it is expected to be formatted as 'Y-m-d H:i:s' and already in the user's and/or system's local time. If it is not, pass the format as the second argument and the timezone (string) as the third.

<div class="clear-right"></div>
#####Form Helper

```php
<?php echo $view['form']->form($form); ?>
```

The form helper, `$view['form']`, works as explained in the [Form](#form-template-helper) section.

<div class="clear-right"></div>
####View Helper Templates
@todo

<div class="clear-right"></div>
####Processing Ajax Content

Caymland provides built in ways to ajaxify links, modals, and forms.

#####Ajax Links
```php
<a href="<?php echo $view['router']->generate('plugin_helloworld_world', array('world' => 'mars')); ?>" data-toggle="ajax" />Mars</a>
```

To ajaxify a link, set the attribute `data-target="ajax"`

#####Ajax Modals
```php
<a href="<?php echo $view['router']->generate('plugin_helloworld_world', array('world' => 'mars')); ?>" 
   data-toggle="ajaxmodal"
   data-target="#CaymlandSharedModal"
   data-header="<?php echo $view['translator']->trans('plugin.helloworld.worlds', array('%world%', 'Mars')); ?>">Mars</a>
```   

Caymland uses Bootstrap modals but Bootstrap lacks an easy way to dynamically retrieve content more than once. Thus, Caymland provides the attribute `data-toggle="ajaxmodal"` to help with this.

`data-target` should be the selector of the model to inject the content into. Caymland has a modal, `#CaymlandSharedModal` on standby to be used for this very purpose.

`data-header` will set the modal's header.

#####Ajax Forms

When using Symfony's [form services](#forms), the form will be auto-ajaxified.

#####Ajax Content Callbacks

```
Caymland.helloWorldDetailsOnLoad = function(container, response) {
    // Manipulate content 
};

Caymland.helloWorldDetailsOnUnload = function(container, response) {
    // Manipulate content 
};
```
Caymland provides a way to execute a javascript function after it injects ajax content into the DOM. This can be useful to activate various JS driven features such as typeaheads, charts, bind events, etc.

To take advantage of this, utilize the caymlandContent element in the passthroughVars array from the controller's [delegateView()](#delegateView($args)). The value of this variable will determine what function should be called after the content has been injected.
 
 For example, the method `Caymland.helloWorldDetailsOnLoad()` will be called for the following: 
 
 <pre class='inline'>
 'passthroughVars' => array(
     'activeLink'    => 'plugin_helloworld_world',
     'route'         => $this->generateUrl('plugin_helloworld_world', array('world' => $world)),
     'caymlandContent' => 'helloWorldDetails'
 )
 </pre>
 
 When the user browses away from the page, `Caymland.helloWorldDetailsOnUnload()` will be called to give opportunity to destroy objects if necessary.
 
 Both the OnLoad and OnUnload methods are passed two arguments.
 
 Argument|Description
 --------|-----------
 container|The selector that was used as the target of the ajax content.
 response|The object from the ajax response, i.e. set in the passthroughVars from the Controller
 
`caymlandContent` should also be set via `$view['slots']` in the [view's](#slots-helper) main page. 
  
<pre class="inline">
$view['slots']->set('caymlandContent', 'helloBundleDetails');
</pre>

Doing so will ensure `Caymland.helloWorldDetailsOnLoad()` is also called when there is a page refresh.
