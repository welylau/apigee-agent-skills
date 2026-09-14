# Antipattern: Use waitForComplete() in JavaScript code

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/wait-for-complete

The [JavaScript policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/javascript-policy) lets you add custom code that executes within the context of an API proxy flow. For example, the custom code in the JavaScript policy can be used to:

- Get and set flow variables
- Execute custom logic and perform fault handling
- Extract data from requests or responses
- Dynamically edit the backend target URL
- Dynamically add or remove headers from a request or a response
- Parse a JSON response

### HTTP Client

A powerful feature of the Javascript policy is the [HTTP client](https://docs.cloud.google.com/apigee/docs/api-platform/reference/javascript-object-model#makingjavascriptcalloutswithhttpclient). The HTTP client (or the `httpClient` object) can be used to make one or multiple calls to backend or external services. The HTTP client is particularly useful when there"s a need to make calls to multiple external services and mash up the responses in a single API.

 **Sample JavaScript Code making a call to backend with httpClient object **

```javascript
var headers = {'X-SOME-HEADER': 'some value' };
var myRequest = new Request("http://www.example.com","GET",headers);
var exchange = httpClient.send(myRequest);
```

 The `httpClient` object exposes two methods `get` and `send ` (`send` is used in the above sample code) to make HTTP requests. Both methods are asynchronous and return an `exchange` object before the actual HTTP request is completed.

 The HTTP requests may take a few seconds to a few minutes. After an HTTP request is made, it's important to know when it is completed, so that the response from the request can be processed. One of the most common ways to determine when the HTTP request is complete is by invoking the `exchange` object's `waitForComplete()` method.

### waitForComplete()

 The `waitForComplete()` method pauses the thread until the HTTP request completes and a response (success/failure) is returned. Then, the response from a backend or external service can be processed.

 **Sample JavaScript code with waitForComplete() **

```javascript
var headers = {'X-SOME-HEADER': 'some value' };
var myRequest = new Request("http://www.example.com","GET",headers);
var exchange = httpClient.send(myRequest);
// Wait for the asynchronous GET request to finish
exchange.waitForComplete();

// Get and Process the response
if (exchange.isSuccess()) {
    var responseObj = exchange.getResponse().content.asJSON;
    return responseObj.access_token;
} else if (exchange.isError()) {
    throw new Error(exchange.getError());
}
```

## Antipattern

 Using `waitForComplete()` after sending an HTTP request in JavaScript code will have performance implications.

 Consider the following JavaScript code that calls `waitForComplete()` after sending an HTTP request.

 **Code for sample.js**

```javascript
// Send the HTTP request
var exchangeObj = httpClient.get("<a href="http://example.com">http://example.com</a>");
// Wait until the request is completed
exchangeObj.waitForComplete();
// Check if the request was successful
if (exchangeObj.isSuccess())  {

    response = exchangeObj.getResponse();
    context.setVariable('example.status', response.status);
} else {
   error = exchangeObj.getError();
   context.setVariable('example.error', 'Woops: ' + error);
}
```

 In this example:

1. The JavaScript code sends an HTTP request to a backend API.
2. It then calls `waitForComplete()` to pause execution until the request completes.
  The `waitForComplete() `API causes the thread that is executing the JavaScript code to be blocked until the backend completes processing the request and responds back.

 There's an upper limit on the number of threads (30%) that can concurrently execute JavaScript code on a Message Processor at any time. After that limit is reached, there will not be any threads available to execute the JavaScript code. So, if there are too many concurrent requests executing the `waitForComplete()` API in the JavaScript code, then subsequent requests will fail with a `500 Internal Server Error` and `Timed out` error message even before the JavaScript policy times out.

 In general, this scenario may occur if the backend takes a long time to process requests or there is high traffic.

## Impact

1. The API requests will fail with `500 Internal Server Error `and with error message `Timed out` when the number of concurrent requests executing `waitForComplete()` in the JavaScript code exceeds the predefined limit.
2. Diagnosing the cause of the issue can be tricky as the JavaScript fails with `Timed out` error even though the time limit for the specific JavaScript policy has not elapsed.

## Best Practice

 Use callbacks in the HTTP client to streamline the callout code and improve performance and avoid using `waitForComplete()` in JavaScript code. This method ensures that the thread executing JavaScript is not blocked until the HTTP request is completed.

 When a callback is used, the thread sends the HTTP requests in the JavaScript code and returns back to the pool. Because the thread is no longer blocked, it is available to handle other requests. After the HTTP request is completed and the callback is ready to be executed, a task will be created and added to the task queue. One of the threads from the pool will execute the callback based on the priority of the task.

 **Sample JavaScript code using Callbacks in httpClient**

```javascript
function onComplete(response,error) {
 // Check if the HTTP request was successful
    if (response) {
      context.setVariable('example.status', response.status);
     } else {
      context.setVariable('example.error', 'Woops: ' + error);
     }
}
// Specify the callback Function as an argument
httpClient.get("<a href="http://example.com">http://example.com</a>", onComplete);
```

## Further reading

- [JavaScript policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/javascript-policy)
- [Making JavaScript callouts with httpClient](https://docs.cloud.google.com/apigee/docs/api-platform/reference/javascript-object-model#makingjavascriptcalloutswithhttpclient)
