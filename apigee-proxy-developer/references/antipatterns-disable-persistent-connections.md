# Antipattern: Disable HTTP persistent (reusable keep-alive) connections

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/disable-persistent-connections

An API proxy is an interface for client applications used to connect with backend services. Apigee provides multiple ways to connect to backend services through an API proxy:

- [TargetEndpoint](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-apis-and-api-proxies) to connect to any HTTP/HTTPs, NodeJS, or Hosted Target services.
- [ServiceCallout policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/service-callout-policy) to invoke any external service pre- or post- invocation of the target server in [TargetEndpoint](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-apis-and-api-proxies).
- Custom code added to the [JavaScript policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/javascript-policy) or [JavaCallout policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/java-callout-policy) to connect to backend services.

### Persistent Connections

 [HTTP persistent connection](https://en.wikipedia.org/wiki/HTTP_persistent_connection), also called HTTP keep-alive or HTTP connection reuse, is a concept that allows a single [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) connection to send and receive multiple [HTTP requests](https://en.wikipedia.org/wiki/HTTP_request)/responses, instead of opening a new connection for every request/response pair.

 Apigee uses persistent connection for communicating with backend services. A connection stays alive for 60 seconds by default. That is, if a connection is idle in the connection pool for more than 60 seconds, then the connection closes.

 The keep alive timeout period is configurable through a property named `keepalive.timeout.millis`, specified in the TargetEndpoint configuration of an API proxy. For example, the keep alive time period can be set to 30 seconds for a specific backend service in the TargetEndpoint.

 In the example below, the `keepalive.timeout.millis` is set to 30 seconds in the TargetEndpoint configuration:

```xml
<!-- /antipatterns/examples/disable-persistent-connections-1.xml -->
<TargetEndpoint name="default">
  <HTTPTargetConnection>
    <URL>http://mocktarget.apigee.net</URL>
    <Properties>
      <Property name="keepalive.timeout.millis">30000</Property>
    </Properties>
  </HTTPTargetConnection>Disable HTTP persistent (Reusable keep-alive) connections
</TargetEndpoint>
```

 In the example above, `keepalive.timeout.millis` controls the keep alive behavior for a specific backend service in an API proxy. There is also a property that controls keep alive behavior for all backend services in all proxies. The `HTTPTransport.keepalive.timeout.millis` is configurable in the Message Processor component. This property also has a default value of 60 seconds. Making any modifications to this property affects the keep alive connection behavior between Apigee and all the backend services in all the API proxies.

## Antipattern

 Disabling persistent (keep alive) connections by setting the property `keepalive.timeout.millis` to 0 in the TargetEndpoint configuration of a specific API Proxy or setting the `HTTPTransport.keepalive.timeout.millis` to 0 on Message Processors is not recommended as it will impact performance.

 In the example below, the TargetEndpoint configuration disables persistent (keep alive) connections for a specific backend service by setting `keepalive.timeout.millis` to 0:

```xml
<!-- /antipatterns/examples/disable-persistent-connections-2.xml -->
<TargetEndpoint name="default">
  <HTTPTargetConnection>
    <URL>http://mocktarget.apigee.net</URL>
    <Properties>
      <Property name="keepalive.timeout.millis">0</Property>
     </Properties>
  </HTTPTargetConnection>
</TargetEndpoint>
```

 If keep alive connections are disabled for one or more backend services, then Apigee must open a new connection for each new request to the target backend service(s). If the backend is HTTPs, Apigee will also perform SSL handshake for each new request, adding to the overall latency of API requests.

## Impact

- Increases the overall response time of API requests as Apigee must open a new connection and perform SSL handshake for every new request.
- Connections may be exhausted under high traffic conditions, as it takes some time to release connections back to the system.

## Best Practice

- Backend services should honor and handle HTTP persistent connection in accordance with HTTP 1.1 standards.
- Backend services should respond with a `Connection:keep-alive` header if they are able to handle persistent (keep alive) connections.
- Backend services should respond with a `Connection:close` header if they are unable to handle persistent connections.

 Implementing this pattern will ensure that Apigee can automatically handle persistent or non-persistent connection with backend services, without requiring changes to the API proxy.

## Further reading

- [HTTP persistent connections](https://en.wikipedia.org/wiki/HTTP_persistent_connection)
- [Keep-Alive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive)
- [Endpoint Properties Reference](https://docs.cloud.google.com/apigee/docs/api-platform/reference/endpoint-properties-reference)
