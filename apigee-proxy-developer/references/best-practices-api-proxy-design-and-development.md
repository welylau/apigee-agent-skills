Best Practices for API Proxy Design and Development
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/best-practices-api-proxy-design-and-development

Summary:
This guide outlines best practices for developing Apigee API proxies, covering design, coding, policy use, logging, monitoring, and debugging.

Key Best Practices:
1. Comments & Documentation: Use inline comments in ProxyEndpoint and TargetEndpoint for readability.
2. Framework-style Coding: Store resources in version control for reuse. Clean up unused policies.
3. Naming Conventions: Match policy name to file name. Use consistent variable naming styles (camelCase or under_score). Name policies by function (e.g., AM-xxx for AssignMessage).
4. Design Considerations:
   - Prefer built-in policies over custom code (JavaScript, Java, Python).
   - Organize Flows with single conditions.
   - Create a default proxy for the "/" basepath.
   - Limit to 3,000 basepaths per environment for optimal performance.
   - Use TargetServers to decouple URLs.
5. CORS: Enable CORS in the request PreFlow of the ProxyEndpoint.
6. Message Payload Size: Default limit is 10MB, configurable to 30MB. Isolate large payload proxies.
7. Fault Handling: Use FaultRules. Use AssignMessage (not RaiseFault) to build fault responses in FaultRules.
8. Response Caching: Only cache successful GET requests. Keep lookup close to client request (PreFlow) and population close to response (PostFlow).
9. Custom Code: JavaScript is preferred over Python and Java for most cases. Use Java only when performance is critical. Avoid making HTTP requests inside scripts; use ServiceCallout instead.
10. Logging: Use a common syslog policy for consistent formatting.
