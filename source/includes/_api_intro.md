# REST API

Caymland provides a REST API to manipulate leads and/or obtain information for various entities of Caymland. 

<aside class="warning">
All Caymland API endpoints require an OAuth1a signtature or OAuth2 access token.
</aside>

## Error Handling

If an OAuth error is encountered, it'll be a JSON encoded array similar to:

<pre class="inline">
{
  "error": "invalid_grant",
  "error_description": "The access token provided has expired."
}
</pre>

If a system error encountered, it'll be a JSON encoded array similar to:

<pre class="inline">
{
    "error": {
        "message": "You do not have access to the requested area/action.",
        "code": 403
    }
}
</pre>

## Caymland version check

In case your API service wants to support several Caymland versions with different features, you might need to check the version of Caymland you communicate with. Since Caymland 2.4.0 the version number is added to all API response headers. The header name is `Caymland-Version`. With Caymland PHP API library you can get the Caymland version like this:

```
// Make any API request:
$api = $this->getContext('contacts');
$response = $api->getList('', 0, 1);

// Get the version number from the response header:
$version = $api->getCaymlandVersion();
```
`$version` will be in a semantic versioning format: `[major].[minor].[patch]`. For example: `2.4.0`. If you'll try it on the latest GitHub version, the version will have `-dev` at the end. Like `2.5.1-dev`.
