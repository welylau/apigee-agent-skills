Message Flow Variable
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/message-variables

Summary:
The `message` flow variable is a contextual object available in all phases of an API proxy flow (request, response, and error flows). It is particularly useful when specific request or response objects go out of scope.

Key Concepts:
1. Scope & Availability: Unlike the `request` and `response` variables, which have limited scope based on the current flow segment, the `message` variable is available globally across all contexts, including error flows.
2. Primary Use Cases:
   - Error Flows: When a proxy enters an error flow, the standard `response` object is out of scope. Attempting to set response headers using `context.setVariable('response.header.X', 'value')` in JavaScript will fail. Instead, use `error` or `message` (e.g., `context.setVariable('message.header.X', 'value')`).
   - PostClientFlow Logging: The `message` variable can be used in a MessageLogging policy within the PostClientFlow to seamlessly log response data for both successful transactions and error conditions.
3. Reusability: Because `message` is valid in both success and error paths, policies (like JavaScript) that reference `message.header.XYZ` can be reused across different flow segments without failing due to scope issues.
4. AssignMessage Policy Exception: The AssignMessage policy automatically handles context switching between request/response and error flows, making it more forgiving than direct JavaScript manipulation.
