## CaymlandJS API Functions

### CaymlandJS.serialize(object)

This method will transform an object properties into a key=value string, concatenating them with an ampersand. It is used when submitting data via CaymlandJS.makeCORSRequest.

```js
var obj = {firstname: "John", lastname: "Doe"};

var serialized = CaymlandJS.serialize(obj);

alert(serialized); // Shows "firstname=John&lastname=Doe"
```

### CaymlandJS.documentReady(functionName|function)

This method will check if the document has finished rendering, then execute the given function. The function argument can be the name of a function or an anonymous function.

```js
function test() {
    alert('test');
}

CaymlandJS.documentReady(test);
```

### CaymlandJS.iterateCollection(collection)(functionName|function)

This method will iterate over the provided collection (array, object, HTMLCollection, etc) using the provided function argument. The function argument can be the name of a function or an anonymous function. The function will receive the collection node and the iteration number as arguments.

```js
var videos = document.getElementsByTagName('video');

// Add a custom data attribute to all videos
CaymlandJS.iterateCollection(videos)(function(node, i) {
    node.dataset.customAttribute = 'test';
});
```

### CaymlandJS.log(arguments)

This method is a lightweight wrapper around `console.log`. It exists because some browsers do not provide this functionality. It takes any number of arguments, logs them, then passes those same arguments to the `console.log` method if it exists.

```js
CaymlandJS.log('Something happened');
```

### CaymlandJS.createCORSRequest(method, url)

This method creates an `XMLHttpRequest`, then checks to see if it supports the `withCredentials` property. If not, we are probably on windows, so it then checks for the existence of `XDomainRequest`, then creates it if found. Finally, it opens then returns the xhr. It can be used to send cross domain requests that include the cookies for the domain. It is used internally within the `CaymlandJS.makeCORSRequest` method.

```js
CaymlandJS.createCORSRequest('GET', 'https://mycaymland.com/dwc/slot1');
```

### CaymlandJS.makeCORSRequest(method, url, data, callbackSuccess, callbackError)

This method uses `CaymlandJS.createCORSRequest` to open a cross domain request to the specified URL, then sets the callbackSuccess and callbackError values appropriately. You may omit either of the callbacks. If you do, the callbacks are replaced with a basic function that uses `CaymlandJS.log(response)` to log the response from the request. The callback methods receive the server response and the xhr object as arguments. If the response is determined to be a JSON string, it is automatically parsed to a JSON object. The data argument will be serialized using `CaymlandJS.serialize(data)`, then sent with the request to the server. All requests made this way have the `X-Requested-With` header set to `XMLHttpRequest`.

```js
CaymlandJS.makeCORSRequest('GET', 'https://mycaymland.com/dwc/slot1', [], function (response, xhr) {
    if (response.success) {
        document.getElementById('slot1').innerHTML = response.content;
    }
});
```

### CaymlandJS.parseTextToJSON(maybeJSON)

This method will take a text string and check to see if it is a valid JSON string. If so, it parses it into a JSON object and returns. If not, then it will simply return the argument passed to it.

```js
var text = '{"firstname": "John", "lastname": "Doe"}';

var json = CaymlandJS.parseTextToJSON(text);

alert(json); // Will show [object Object]

var text = 'not valid json';

var json = CaymlandJS.parseTextToJSON(text);

alert(json); // Will show 'not valid json'
```

### CaymlandJS.insertScript(scriptUrl)

This method will insert a script tag with the provided URL in the head of your document, before other scripts.

```js
CaymlandJS.insertScript('http://google.com/ga.js');
```
