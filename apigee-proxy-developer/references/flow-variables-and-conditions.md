Conditions with Flow Variables
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/flow-variables-and-conditions

Summary:
Apigee supports conditional statements in Flows, Policies, Steps, and RouteRules to enable dynamic API behavior without writing code. Conditions evaluate flow variables to determine execution path.

Key Concepts:
1. Condition Syntax: `<Condition>VARIABLE_NAME OPERATOR "VALUE"</Condition>` (e.g., `<Condition>request.verb = "GET"</Condition>`).
2. Operators:
   - Standard: `=`, `!=`, `>`, `equals`, `notequals`, `greaterthan`.
   - Pattern Matching (`Matches` or `~`): Supports literal matching and simple wildcards (`*` matches zero or more characters).
   - Regex (`JavaRegex` or `~~`): Uses Java regular expression syntax for complex matching. Case-sensitive by default.
   - Path Matching (`MatchesPath` or `~/`): Specifically designed for URI path fragments. Evaluates paths as discrete segments.
3. MatchesPath Wildcards:
   - Single asterisk (`*`): Matches exactly one path segment. (e.g., `/animals/*` matches `/animals/cats` but not `/animals/cats/wild`).
   - Double asterisk (`**`): Matches one or more path segments. (e.g., `/animals/**` matches both `/animals/cats` and `/animals/cats/wild`).
4. Context Variables: Conditions rely on dotted notation variables (e.g., `request.header.Content-type`) provided by the proxy runtime context.
5. Mapping Resources: A common pattern is to combine `proxy.pathsuffix` and `request.verb` to map incoming requests to specific backend resources or specialized policies.
