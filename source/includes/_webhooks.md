# Webhooks

Webhook is a universal way how to send data about leads, pages, forms and events. The data is sent in real-time when an action occurs so the system which listens form Caymland webhook data can process them without the need for a periodic scanning if Caymland has some new data.

## Available webhook actions

Caymland can send webhook payload on these actions:

- Email open
- Form submit
- Lead delete
- Lead point change
- Lead update
- Lead create
- Page hit

## The webhook workflow

The example workflow below describes a real-life workflow to get an idea how the webhooks can be used. Let's imagine we have a project management system (PMS) and we want to create a new issue when a lead submits a form.

1. A lead submits a Caymland form.
2. Caymland saves the form.
3. Caymland checks if there is a webhook with the *Form submit* event. If there is, Caymland sends the data to the URL address defined in the webhook.
4. PMS receives the data about the form submission and creates a new issue from it.

## Create a webhook

It is possible to create multiple different webhooks. That will allow you to send different information to different apps/scripts.

1. Open the right hand side menu (click the cog icon in the top right corner) and select *Webhooks*.
2. Create a new webhook.
3. Fill in a *Name*, *Webhook POST Url* (see the next paragraph if you don't have one) and select which *Webhook Events* should trigger this webhook.

## Test a webhook

The easiest way how to test a webhook payload is to use a service like [RequestBin](http://requestb.in/). RequestBin lets you create a URL which you can set as the `Webhook POST Url` in Caymland. Then click the *Apply* button to save it and then click the *Send Test Payload* button. That will send a test payload data to RequestBin and you will be able to see it at your RequestBin.

When you have created your testing webhook, you can test the real data it sends when a defined action is triggered.

## Immediate or queued webhooks

There is an option to queue webhooks for background execution. The reason behind it is that every time an action like contact update happens which has the webhook listener attached to it, the action has to wait for the webhook response untill the webhook response returns or when it times out after 10 senconds. So it is up to the webhook reciever how fast the contact update is.

This lag can be more visible when you do a CSV import. It may be slow when it is waiting a second or two for webhook response for every imported contact.

If you want to avoid this lag, configure the webhook queue in the configuration and add this command to your cron tab: `app/console caymland:webhooks:process`. This way every time the webhook is triggered, the action is queued as a new row into database, so it is much faster and then the command will make the requests which may take some time. The caveat to this optimisation is that the webhooks are not fired every time the action happens, but every time the command runs.

## Queued event order

Caymland will send several events in one webhook if they happen before the queue trigger command runs. Caymland's default order of those events is chronological. But Zapier integration which use webhooks havily requires reverse chronological order. Thereofore the new option to configure the order was added to webhooks as well as to the global configuration. Webhook configuration overwrites the global configuration, but if not set, the global configuration order value is applied.

## Example webhook script

If you need an idea about how to receive Caymland webhook data in your app, this script can be used as a starting point. The script will log the request and return a PHP object of the payload. Place this script on a publicly accessible URL (i.e. `http://yourwebsite.com/webhookTest.php), then fill in the Caymland *Webhook POST Url* to this script.

 ```php
 <?php
// webhookTest.php

/**
 * A helper class to log and get the Caymland webhook request
 */
class webhookTest {
    /**
     * Log a message to a file
     *
     * @param  mixed $message
     * @param  string $type [info|error|request]
     */
    public function log($message, $type = 'info')
    {
        $prefix = 'webhookLog_';
        $file = $type . '.log';
        $date = new DateTime();
        error_log($date->format('Y-m-d H:i:s') . ' ' . $message . "\n\n", 3, $prefix . $file);
    }
    /**
     * Get the request JSON object and log the request
     *
     * @return object
     */
    public function getRequest()
    {
        $rawData = file_get_contents("php://input");
        $this->log($rawData, 'request');
        return json_decode($rawData);
    }
}
$webhook = new webhookTest;
$requestData = $webhook->getRequest();
// @todo Process the $requestData as needed
```
If you'd like to extend the webhook functionality with your plugin, read more in [the plugin docs](#extending-webhooks).
