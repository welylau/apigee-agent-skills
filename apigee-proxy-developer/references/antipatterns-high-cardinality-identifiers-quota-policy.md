# Antipattern: Use high-cardinality identifiers in Quota policies

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/high-cardinality-identifiers-quota-policy

The [Quota policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/quota-policy) is used to count the number of requests received by an API proxy. This capability enables API providers to enforce limits on the number of API calls made by apps over an interval of time.

 The Quota policy can include an [`identifier`](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/quota-policy#identifier) element that identifies the quota "bucket" in which each request is counted.

## Antipattern

 When using the Quota policy, do not use high-cardinality identifiers.

 Cardinality refers to the number of unique data values in a set. An identifier with high cardinality has a large number of distinct possible values. High-cardinality identifiers include unique request IDs or session IDs that change with every API call.

 Using high-cardinality identifiers can significantly undermine the effectiveness of your quota enforcement.

## Impact

 Using high-cardinality identifiers for your quota policy's identifier element causes:

-  **Ineffective Quota Enforcement:** Each unique identifier is treated as a separate counter. If every request has a new, unique ID, your quota system essentially creates a new "bucket" for every API call. This means your overall quota limit is measured against individual, single-use counters rather than by actual groups of requests you want to limit, rendering the policy useless for traffic control.
-  **Increased Resource Consumption:** Generating and managing a massive number of unique quota counters places unnecessary strain on the Apigee platform, leading to increased resource usage and potential performance issues.
-  **Monitoring Challenges:** It becomes difficult to monitor and understand actual API consumption trends when the data is fragmented across large numbers of unique identifiers. You lose the ability to see which applications, developers, or products are consuming your API resources.

## Best practice

 Choose identifiers with low to medium cardinality that also represent a stable and meaningful grouping for quota enforcement. These help you manage API usage effectively and gain insights into your traffic. Examples include:

- developer.app.name
- client_id
- apiproduct.name

 With appropriate identifiers, your Quota policy can more effectively manage API traffic, prevent unintended overages, and provide clear insights into usage patterns.
