Handling Faults
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/fault-handling

Summary:
Apigee allows developers to customize error responses using FaultRules and DefaultFaultRule. When a policy fails or a system error occurs, the proxy enters an error state and executes designated fault-handling logic instead of the normal flow.

Key Concepts:
1. FaultRules and DefaultFaultRule:
   - `<FaultRules>`: Contains one or more `<FaultRule>` elements targeting specific errors.
   - `<DefaultFaultRule>`: Executes if no specific FaultRule matches, providing a fallback error response.
2. Condition-Based Execution: Each `<FaultRule>` should include a `<Condition>` (e.g., `fault.name = "QuotaViolation"`) to ensure it only triggers for the intended error.
3. Evaluation Order:
   - ProxyEndpoint: Evaluated from **bottom to top**. The first rule that evaluates to true is executed.
   - TargetEndpoint: Evaluated from **top to bottom**. The first rule that evaluates to true is executed.
4. The `continueOnError` Attribute: A property on policies. If set to `true`, a policy failure will not trigger an error state, bypassing FaultRules and continuing the normal flow.
5. Placement Strategy: Define FaultRules in the `ProxyEndpoint` for client-facing errors and in the `TargetEndpoint` for backend/target communication errors.
6. RaiseFault Policy: A specific policy used to explicitly generate custom errors and force the proxy into the error flow.
