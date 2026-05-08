Conditional Flows (Flow Configurations)
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/flow-configurations

Summary:
Conditional flows enable policies to be executed only when specific conditions evaluate to true, allowing for dynamic behavior based on the state of the API transaction.

Key Concepts:
1. Execution Rule: During the processing of a request or response, only one conditional flow is executed per segment (e.g., ProxyEndpoint request pipeline)—the first flow whose condition evaluates to true.
2. Conditions: Conditions are constructed using flow variables (e.g., `request.verb`, `proxy.pathsuffix`).
3. Examples:
   - Matching HTTP Method: `<Condition>request.verb="GET"</Condition>`
   - Matching Path Suffix: `<Condition>(proxy.pathsuffix MatchesPath "/reports")</Condition>`
4. Configuration:
   - Conditional flows are defined within the `<Flows>` element of a ProxyEndpoint or TargetEndpoint.
   - Policies are attached as steps within the `<Request>` or `<Response>` elements of the flow.
   - Operators like `&&` (AND) and `||` (OR) can be used to create complex conditions, with standard precedence rules (parentheses can be used to group).
