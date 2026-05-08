Chaining API Proxies Together
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/connecting-proxies-other-proxies

Summary:
Proxy chaining allows one API proxy to act as the target endpoint for another. This local connection avoids network overhead (like load balancers and routers), improving performance for composite or modular services.

Key Concepts:
1. Definition: Specifying a second proxy as the local target of a first proxy using `<LocalTargetConnection>` instead of `<HTTPTargetConnection>`.
2. Benefits: Reduces network hops and latency by keeping communication internal to the message processor.
3. Connection Methods:
   - By Proxy Name: Specify `<APIProxy>` and `<ProxyEndpoint>` names. Ideal when proxies are developed together.
   - By Path: Specify the target proxy's endpoint `<Path>`. Useful when the target proxy name is unknown or managed by another team. Supports dynamic assignment via message templates.
4. Security Best Practice: Since chained proxies are typically exposed publicly by default, secure target proxies against direct external calls by validating that the client IP is local (e.g., using the AccessControl policy).
