What are Flows
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/what-are-flows

Summary:
Flows are the basic building blocks of API proxies, enabling developers to configure the sequence in which policies and code are executed to program API behavior.

Key Concepts:
1. Flow Structure: Flows are sequential stages along the request and response processing paths in both ProxyEndpoint and TargetEndpoint.
2. Types of Flows:
   - PreFlow: Executes first. Ideal for security (authentication), traffic management (quota, spike arrest), and initial request preparation. Always executes.
   - Conditional Flows: Execute between PreFlow and PostFlow. They allow for branching logic based on conditions (e.g., HTTP verb, path). Only the first flow with a true condition executes in this segment.
   - PostFlow: Executes after conditional flows. Ideal for message transformation, setting response headers, and non-terminal logging.
   - PostClientFlow: Executes after the response is sent back to the client. It only supports MessageLogging, ServiceCallout, and FlowCallout policies (if the shared flow meets the same criteria). Ideal for final logging and analytics.
3. Logic Execution: Policies are attached as "Steps" within a flow and execute in the order they are listed.
4. Debugging: The Debug tool visualizes the execution path but does not explicitly demarcate the boundaries between PreFlow, conditional flows, and PostFlow.
