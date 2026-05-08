Shared Flows
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/shared-flows

Summary:
Shared flows allow developers to combine policies and resources into a reusable sequence that can be executed from multiple API proxies or even other shared flows. They promote consistency, reduce development time, and simplify code management.

Key Concepts:
1. Definition: A shared flow is a reusable sequence of conditional steps. Unlike an API proxy, it has no endpoint and cannot be called directly by clients.
2. Scope: Must be in the same organization as the consuming API proxy or shared flow.
3. Usage Methods:
   - FlowCallout Policy: Used within an API proxy or shared flow to explicitly invoke the shared flow at a specific point.
   - Flow Hooks: Used to attach a shared flow globally to execute at specific points (e.g., before a proxy request, after a target response) for all proxies in an environment.
4. Development and Testing:
   - Developed similarly to API proxies (sequence of steps and policies).
   - Cannot be tested directly; must be invoked through a test API proxy that calls the shared flow.
5. Deployment:
   - Must be deployed to the same environment as the consuming proxies.
   - Best Practice: Deploy the shared flow *before* the consuming API proxies to ensure dependencies are resolved at deployment time.
