Understanding Routes
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-routes

Summary:
A route determines the path of a request from the ProxyEndpoint to the TargetEndpoint, involving the inbound URL and the backend service URL.

Key Concepts:
1. Inbound URL Structure: `https://{hostname}/{basepath}/{resource}`
   - Hostname: Domain mapped to an environment group.
   - BasePath: Unique identifier for the API proxy in an environment (defined in ProxyEndpoint).
   - Resource: Specific path mapped to conditional flows.
2. Route Rules (<RouteRule>):
   - Evaluated after ProxyEndpoint request PreFlow, Conditional Flows, and PostFlow policies.
   - Determines where to send the request.
3. Target Types:
   - Direct URL: Calls a backend directly (e.g., `<URL>http://example.com</URL>`). No TargetEndpoint policies can be applied.
   - Single Target: Routes all traffic to a named TargetEndpoint (e.g., `<TargetEndpoint>default</TargetEndpoint>`).
   - Conditional Targets: Routes based on conditions (headers, query params, variables). Rules are evaluated top-down; the first match wins. The default (unconditional) rule must be last.
   - Null Route: Does not forward to a target (e.g., `<RouteRule name="GoNowhere"/>`). Useful when Apigee generates the response directly (e.g., via JavaScript).
