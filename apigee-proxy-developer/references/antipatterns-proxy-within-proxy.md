# Antipattern: Invoking a proxy within a proxy using custom code or as a target

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/proxy-within-proxy

Apigee lets you invoke one API proxy from another API proxy. This feature is useful especially if you have an API proxy that contains reusable code that can be used by other API proxies.

## Antipattern

Invoking one API proxy from another either using `HTTPTargetConnection` in the target endpoint or custom JavaScript code results in additional network hop.

### Invoke proxy 2 from proxy 1 using `HTTPTargetConnection`

The following code sample invokes proxy 2 from proxy 1 using `HTTPTargetConnection`:

```xml
<!-- /antipatterns/examples/2-1.xml -->
<HTTPTargetConnection>
  <URL>https://api-test.example.com/proxy2</URL>
</HTTPTargetConnection>
```

### Invoke proxy 2 from proxy 1 from the JavaScript code

The next code sample invokes proxy 2 from proxy 1 using JavaScript:

```xml
<!-- /antipatterns/examples/2-2.xml -->
var response = httpClient.send('https://api-test.example.com/proxy2);
response.waitForComplete();
```

### Code Flow

To understand why this has an inherent disadvantage, we need to understand the route a request takes as illustrated by the diagram below:

![1) Client makes request to Proxy 1, 2) Request from Proxy 1 to Proxy 2 incurs network hop, 3) Request from Proxy 2 to target.](https://docs.cloud.google.com/apigee/docs/api-platform/images/antipatterns/proxies-requests.png)
 ***Figure 1**: Code flow*

As depicted in the diagram, a request traverses multiple distributed components, including the Router and the Message Processor.

In the code samples above, invoking proxy 2 from proxy 1 means that the request has to be routed through the traditional route (Router > MP) at runtime. This would be akin to invoking an API from a client thereby making multiple network hops that add to the latency. These hops are unnecessary considering that proxy 1 request has already *reached* the MP.

## Impact

Invoking one API proxy from another API proxy incurs unnecessary network hops, that is the request has to be passed on from one Message Processor to another Message Processor.

## Best practice

- Use the [proxy chaining](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/connecting-proxies-other-proxies) feature for invoking one API Proxy from another. Proxy chaining is more efficient as it uses local connection to reference the target endpoint (another API Proxy).
  The code sample shows proxy chaining using `LocalTargetConnection` in your endpoint definition:

  ```xml
  <!-- /antipatterns/examples/2-3.xml -->
  <LocalTargetConnection>
    <APIProxy>proxy2</APIProxy>
    <ProxyEndpoint>default</ProxyEndpoint>
  </LocalTargetConnection>
  ```

  The invoked API Proxy gets executed within the same Message Processor; as a result, it avoids the network hop as shown in the following figure:

![1) Client makes request to Proxy 1, 2) Request from Proxy 1 to Proxy 2 made via psuedo-local call, 3) Request from Proxy 2 to target.](https://docs.cloud.google.com/apigee/docs/api-platform/images/antipatterns/proxy-chaining.png)
 ***Figure 2**: Code flow with proxy chaining*

## Further reading

- [Proxy chaining](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/connecting-proxies-other-proxies)
