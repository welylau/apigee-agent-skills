Using Flow Variables
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/introduction-flow-variables

Summary:
Flow variables are objects used to maintain state during API transactions processed by Apigee. They enable policies and conditional flows to access and manipulate transaction data.

Key Concepts:
1. Purpose: Store information about requests, responses, system state, and policy execution results.
2. Types:
   - Built-in: Automatically created and populated by Apigee (e.g., `client.ip`, `request.verb`, `system.time`).
   - Custom: Created by developers using policies (like AssignMessage or ExtractVariables) or within code (JavaScript/Java).
3. Scope: A variable is only accessible after it has been instantiated in the flow. Accessing a variable before its scope results in a NULL value.
   - Proxy request scope: Available throughout the transaction.
   - Target request scope: Available only after entering the TargetEndpoint request flow.
4. Usage:
   - In Policies: Enclosed in curly braces (e.g., `{client.ip}`) to inject values into configurations.
   - In Conditional Flows: Used without braces (e.g., `request.verb = "POST"`) to control flow execution.
   - In JavaScript: Accessed via getter/setter methods on objects like `context` (e.g., `context.getVariable('system.time.year')`).
5. Naming: Follows dot-notation (e.g., `request.content`, `system.time`). Common prefixes include `request`, `response`, `system`, and `target`.
