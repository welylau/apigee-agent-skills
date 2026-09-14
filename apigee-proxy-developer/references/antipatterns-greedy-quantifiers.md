# Antipattern: Use greedy quantifiers in the RegularExpressionProtection policy

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/greedy-quantifiers

The [RegularExpressionProtection policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/regular-expression-protection) defines regular expressions that are evaluated at runtime on input parameters or flow variables. You typically use this policy to protect against content threats like SQL orJavaScript injection, or to check against malformed request parameters like email addresses or URLs.

The regular expressions can be defined for request paths, query parameters, form parameters, headers, XML elements (in an XML payload defined using XPath), JSON object attributes (in a JSON payload defined using JSONPath).

The following example RegularExpressionProtection policy protects the backend from SQL injection attacks:

```xml
<!-- /antipatterns/examples/greedy-1.xml -->
<RegularExpressionProtection async="false" continueOnError="false" enabled="true"
  name="RegexProtection">
    <DisplayName>RegexProtection</DisplayName>
    <Properties/>
    <Source>request</Source>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <QueryParam name="query">
      <Pattern>[\s]*(?i)((delete)|(exec)|(drop\s*table)|
        (insert)|(shutdown)|(update)|(\bor\b))</Pattern>
    </QueryParam>
</RegularExpressionProtection>
```

## Antipattern

The default quantifiers (`*`, `+`, and `?`) are greedy in nature: they start to match with the longest possible sequence. When no match is found, they backtrack gradually to try to match the pattern. If the resultant string matching the pattern is very short, then using greedy quantifiers can take more time than necessary. This is especially true if the payload is large (in the tens or hundreds of KBs).

The following example expression uses multiple instances of `.*`, which are greedy operators:

```xml
<Pattern>.*Exception in thread.*</Pattern>
```

In this example, the [RegularExpressionProtection policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/regular-expression-protection) first tries to match the longest possible sequence—the entire string. If no match is found, the policy then backtracks gradually. If the matching string is close to the start or middle of the payload, then using a greedy quantifier like `.*` can take a lot more time and processing power than reluctant qualifiers like `.*?` or (less commonly) possessive quantifiers like `.*+`.

Reluctant quantifiers (like `X*?`, `X+?`, `X??`) start by trying to match a single character from the beginning of the payload and gradually add characters. Possessive quantifiers (like `X?+`, `X*+`, `X++`) try to match the entire payload only once.

Given the following sample text for the above pattern:

```text
Hello this is a sample text with Exception in thread
with lot of text after the Exception text.
```

Using the greedy `.*` is non-performant in this case. The pattern `.*Exception in thread.*` takes 141 steps to match. If you used the pattern `.*?Exception in thread.*` (which uses a reluctant quantifier) instead, the result would be only 55 steps.

## Impact

Using greedy quantifiers like wildcards (`*`) with the [RegularExpressionProtection policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/regular-expression-protection) can lead to:

- An increase in overall latency for API requests for a moderate payload size (up to 1MB)
- Longer time to complete the execution of the RegularExpressionProtection policy
- API requests with large payloads (>1MB) failing with 504 Gateway Timeout Errors if the predefined timeout period elapses on the Apigee Router
- High CPU utilization on Message Processors due to large amount of processing which can further impact other API requests

## Best practice

- Avoid using greedy quantifiers like `.*` in regular expressions with the [RegularExpressionProtection policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/regular-expression-protection). Instead, use reluctant quantifiers like `.*?` or possessive quantifiers like `.*+` (less commonly) wherever possible.

## Further reading

- [Regular Expression Quantifiers](https://docs.oracle.com/javase/tutorial/essential/regex/quant.html)
- [RegularExpressionProtection policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/regular-expression-protection)
