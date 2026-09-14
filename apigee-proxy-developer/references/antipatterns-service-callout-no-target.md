# Antipattern: Use Service Callout Policy to invoke a backend service in a No Target API Proxy

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/service-callout-no-target

An API proxy is a managed facade for backend services. A basic API proxy configuration consists of a [ProxyEndpoint](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-apis-and-api-proxies) (defining the URL of the API proxy) and a [TargetEndpoint](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-apis-and-api-proxies) (defining the URL of the backend service).

 Apigee offers a lot of flexibility for building sophisticated behavior on top of this pattern. For example, you can add policies to control the way the API processes a client request before sending it to the backend service, or manipulate the response received from the backend service before forwarding it to the client. You can invoke other services using [service callout policies](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/service-callout-policy), add custom behavior by adding [Javascript code](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/javascript-policy), and even create an API proxy that doesn't invoke a backend service.

## Antipattern

 Using service callouts to invoke a backend service in an API proxy with no routes to a target endpoint is technically feasible, but results in the loss of analytics data about the performance of the external service.

 An API proxy that does not contain target routes can be useful in cases where you do not need to forward the request message to the TargetEndpoint. Instead, the ProxyEndpoint performs all necessary processing. For example, the ProxyEndpoint could retrieve data from a lookup to the API service's key/value store and return the response without invoking a backend service.

 You can define a [null Route](https://docs.cloud.google.com/apigee/docs/api-platform/reference/api-proxy-configuration-reference#proxyendpoint-nullroutes) in an API proxy, as shown here:

```xml
<RouteRule name="noroute"/>
```

 A proxy using a null route is a "no target" proxy, because it does not invoke a target backend service.

 It is technically possible to add a service callout to a no target proxy to invoke an external service, as shown in the example below:

```xml
<!-- /antipatterns/examples/service-callout-no-target-1.xml -->
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
    <Description/>
    <FaultRules/>
    <PreFlow name="PreFlow">
        <Request>
            <Step>
                <Name>ServiceCallout-InvokeBackend</Name>
            </Step>
        </Request>
        <Response/>
    </PreFlow>
    <PostFlow name="PostFlow">
        <Request/>
        <Response/>
    </PostFlow>
    <Flows/>
    <HTTPProxyConnection>
        <BasePath>/no-target-proxy</BasePath>
        <Properties/>
        <VirtualHost>secure</VirtualHost>
    </HTTPProxyConnection>
    <RouteRule name="noroute"/>
</ProxyEndpoint>
```

 However, the proxy cannot provide analytics information about the external service behavior (such as processing time or error rates), making it difficult to assess the performance of the external service.

## Impact

- Analytics information on the interaction with the external service ( error codes, response time, target performance, etc.) is unavailable
- Any specific logic required before or after invoking the service callout is included as part of the overall proxy logic, making it harder to understand and reuse.

## Best Practice

 If an API proxy interacts with only a single external service, the proxy should follow the basic design pattern, where the backend service is defined as the target endpoint of the API proxy. A proxy with no routing rules to a target endpoint should not invoke a backend service using the ServiceCallout policy.

 The following proxy configuration implements the same behavior as the example above, but follows best practices:

```xml
<!-- /antipatterns/examples/service-callout-no-target-2.xml -->
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
    <Description/>
    <FaultRules/>
    <PreFlow name="PreFlow">
        <Request/>
        <Response/>
    </PreFlow>
    <PostFlow name="PostFlow">
        <Request/>
        <Response/>
    </PostFlow>
    <Flows/>
    <HTTPProxyConnection>
        <BasePath>/simple-proxy-with-route-to-backend</BasePath>
        <Properties/>
        <VirtualHost>secure</VirtualHost>
    </HTTPProxyConnection>
    <RouteRule name="default">
        <TargetEndpoint>default</TargetEndpoint>
    </RouteRule>
</ProxyEndpoint>
```

 Use Service callouts to support mashup scenarios, where you want to invoke external services before or after invoking the target endpoint. Service callouts are not meant to replace target endpoint invocation.

## Further reading

- [Understanding APIs and API proxies](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-apis-and-api-proxies)
- [How to configure Route Rules](https://docs.cloud.google.com/apigee/docs/api-platform/reference/api-proxy-configuration-reference#proxyendpoint-howtoconfigurerouterules)
- [Null Routes](https://docs.cloud.google.com/apigee/docs/api-platform/reference/api-proxy-configuration-reference#proxyendpoint-nullroutes)
- [Service Callout policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/service-callout-policy.html)
