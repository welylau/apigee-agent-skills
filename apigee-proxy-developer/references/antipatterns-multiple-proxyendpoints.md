# Antipattern: Define multiple ProxyEndpoints in an API proxy

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/multiple-proxyendpoints

The ProxyEndpoint configuration defines the way client apps consume the APIs through Apigee. The ProxyEndpoint defines the URL of the API proxy and how a proxy behaves: which policies to apply and which target endpoints to route to, and the conditions that need to be met for these policies or route rules to be executed.

In short, the ProxyEndpoint configuration defines all that needs to be done to implement an API.

## Antipattern

An API proxy can have one or more proxy endpoints. Defining multiple ProxyEndpoints is an easy and simple mechanism to implement multiple APIs in a single proxy. This lets you reuse policies and/or business logic before and after the invocation of a TargetEndpoint.

On the other hand, when defining multiple ProxyEndpoints in a single API proxy, you end up conceptually combining many unrelated APIs into a single artifact. It makes the API proxies harder to read, understand, debug, and maintain. This defeats the main philosophy of API proxies: making it easy for developers to create and maintain APIs.

## Impact

Multiple ProxyEndpoints in an API proxy can:

- Make it hard for developers to understand and maintain the API proxy.
- Obfuscate analytics. By default, analytics data is aggregated at the proxy level. There is no breakdown of metrics by proxy endpoint unless you create custom reports.
- Make it difficult to troubleshoot problems with API proxies.

## Best practice

When you are implementing a new API proxy or redesigning an existing API proxy, use the following best practices:

1. Implement one API proxy with a single ProxyEndpoint.
2. If there are multiple APIs that share common target server and/or require the same logic pre- or post-invocation of the target server, consider using shared flows to implement such logic in different API proxies.
3. If there are multiple APIs that share a common starting base path, but differ in the suffix, use conditional flows in a single ProxyEndpoint.
4. If there exists an API proxy with multiple ProxyEndpoints and if there are no issues with it, then there is no need to take any action.

Using one ProxyEndpoint per API proxy leads to:

1. Simpler, easier to maintain proxies
2. Better information in Analytics, such as proxy performance and target response time, will be reported separately instead of rolled up for all ProxyEndpoints
3. Faster troubleshooting and issue resolution

## Further reading

- [API Proxy Configuration Reference](https://docs.cloud.google.com/apigee/docs/api-platform/reference/api-proxy-configuration-reference)
- [Reusable shared flows](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/shared-flows)
