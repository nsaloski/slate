# CaymlandJS API

Caymland provides a means for plugins to inject custom JavaScript into mtc.js, the PHP generated script that manages Caymland's tracking pixel, dynamic web content, etc. mtc.js is embedded in 3rd party websites to manage communication between those and Caymland.

## mtc.js

```php
<?php
// plugins/HelloWorldPlugin/Event/BuildJsSubscriber.php

namespace CaymlandPlugin\HelloWorldBundle\EventListener;

use Caymland\CoreBundle\CoreEvents;
use Caymland\CoreBundle\Event\BuildJsEvent;

/**
 * Class BuildJsSubscriber
 */
class BuildJsSubscriber extends CommonSubscriber
{
    /**
     * @return array
     */
    public static function getSubscribedEvents()
    {
        return array(
            CoreEvents::BUILD_CAYMLAND_JS => array('onBuildJs', 0)
        );
    }

    /**
     * @param BuildJsEvent $event
     *
     * @return void
     */
    public function onBuildJs(BuildJsEvent $event)
    {
        $js = <<<JS
CaymlandJS.documentReady(function() {
    // Determine what planet this is coming from
});
JS;
        $event->appendJs($js, 'HelloWorld');
    }
}
```

To inject custom Javascript into mtc.js, use an [event listener](#events) for the `CoreEvents::BUILD_CAYMLAND_JS` event. This event receives a `Caymland\CoreBundle\Event\BuildJsEvent` object where `$event->appendJs($js, $sectionName);` can be used to inject the script's code.

<aside class="warning">
Only native Javascript or <a href="#caymlandjs-api-functions">CaymlandJs API functions</a> should be used since jQuery and other libraries are not guaranteed to be available in 3rd party websites.
</aside>

## JS Form Processing Hooks

```js
if (typeof CaymlandFormCallback == 'undefined') {
    var CaymlandFormCallback = {};
}
CaymlandFormCallback['replaceWithFormName'] = {
    onValidateEnd: function (formValid) {
         // before form submit
    },
    onResponse: function (response) { 
         // after form submit
    }
};

```

If you wish execute additional code to execute as form is being processed create a `CaymlandFormCallback` object.  In the example code replace `replaceWithFormName` with the name of your form. 

`onValidateEnd` and `onResponse` are actions called by `Form.customCallbackHandler`. 

### onValidate()

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onValidate: function () {
        // before form validation
        var formIsGood = True;
        var dontUpdate = False;
        if(dontUpdate){
            return null;
        }else if(formIsGood){
            return True;
        }else if(!formIsGood){
            return False;
        }
    },
};
```

Called before default form validation and can be used to override default form validation.

Return `True` to skip the default form validation continue with form processing. Return `False` to skip the default form validation and prevent the form submission.  Return `null` to execute default form validation.

### onValidateStart()

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onValidateStart: function () {
         // before default validation
    },
};
```

Called at the beginning of the default form validation.  Receives no values and return value is not required and not processed.

<aside class="warning">
onValidateStart may not be executed if the default form validation is handled during the `onValidate` callback
</aside>

### onValidateEnd(formValid)

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onValidateEnd: function (formValid) {
         // before form submit
    },
};
```

Called after all form validation is complete (default and/or `onValidate` callback) and before the form is submitted.  Receives `formValid` to determine if form is valid.  Return value is not required and not processed.

### onErrorMark(callbackData)

```js
var callbackData = {
    containerId: containerId,
    valid: valid,
    validationMessage: callbackValidationMessage
};

CaymlandFormCallback['replaceWithFormName'] = {
    onErrorMark: function (callbackData) {
         // called during error marking
    },
};
```

Called during error marking.  Receives a `callbackData` object. Return `True` to skip the default error marking.

### onErrorClear(containerId)

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onErrorClear: function (containerId) {
         // called to clear an existing error
    },
};
```

Called to clear an existing error.  Receives `containerId` with the id of the element containing the error. Return `True` to skip the default error clearing.

### onResponse(response)

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onResponse: function (response) {
         // called to process the response to the form submission
    },
};
```

Called prior to default form submission response processing.  Receives `response` containing the form submission response. Return `True` to skip the default form submission response processing.

### onResponseStart(response)

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onResponseStart: function (response) {
         // called to process the response to the form submission
    },
};
```

Called at the beginning default form submission response processing.  Receives `response` containing the form submission response. Return value is not required and not processed.

<aside class="warning">
onResponseStart may not be executed if the default response processing is handled during the `onResponse` callback
</aside>

### onResponseEnd(response)

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onResponseEnd: function (response) {
         // called to process the response to the form submission
    },
};
```

Called at the end default form submission response processing.  Receives `response` containing the form submission response. Return value is not required and not processed.

<aside class="warning">
onResponseEnd may not be executed if the default response processing is handled during the `onResponse` callback
</aside>

### onMessageSet(messageObject)

```js
var messageObject = {
    message: message,
    type: type
};

CaymlandFormCallback['replaceWithFormName'] = {
    onErrorMark: function (messageObject) {
         // called prior to default message insertion
    },
};
```

Called prior to default message insertion.  Receives a `messageObject` containing the message and message type. Return `True` to skip the default message insertion.

### onSubmitButtonDisable(messageObject)

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onErrorMark: function (messageObject) {
         // called prior to default message insertion
    },
};
```

Called prior to default disabling of the submit button.  Receives no values. Return `True` to skip the default disabling of the submit button.

### onSubmitButtonEnable()

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onErrorMark: function (messageObject) {
         // called prior to default message insertion
    },
};
```

Called prior to default enabling of the submit button.  Receives no values. Return `True` to skip the default enabling of the submit button.

### onShowNextPage()

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onShowNextPage: function (pageNumber) {
         // called prior to going to the next page
    },
};
```

Called prior to going to the next page in the form. Useful to adjust the DOM prior to making the page visible.


### onShowPreviousPage()

```js
CaymlandFormCallback['replaceWithFormName'] = {
    onShowPreviousPage: function (pageNumber) {
         // called prior to going back to previous page
    },
};
```

Called prior to going back to a previous page in the form. Useful to adjust the DOM prior to making the page visible.


