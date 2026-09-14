# Antipattern: Configure non-distributed quota

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/non-distributed-quota

Apigee provides the ability to configure the number of allowed requests to an API Proxy for a specific period of time using the [Quota policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/quota-policy).

## Antipattern

An API proxy request can be served by one or more distributed Apigee components called *Message Processors*. If there are multiple Message Processors configured for serving API requests, then the quota will likely be exceeded because each Message Processor keeps it's own "count" of the requests it processes.

Let's explain this with the help of an example. Consider the following [Quota policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/quota-policy) for an API proxy:

```xml
<!-- /antipatterns/examples/1-6.xml -->
<Quota name="CheckTrafficQuota">
  <Interval>1</Interval>
  <TimeUnit>hour</TimeUnit>
  <Allow count="100"/>
</Quota>
```

The above configuration should allow a total of 100 requests per hour.

In practice, however, when multiple message processors are serving the API requests, the following happens:

![Diagram](https://docs.cloud.google.com/apigee/docs/api-platform/images/antipatterns/request-to-mps.png)

In the above illustration:

- The quota policy is configured to allow 100 requests per hour.
- The requests to the API Proxy are being served by two Message Processors.
- Each Message Processor maintains its own quota count variable, `quota_count_mp1` and `quota_count_mp2`, to track the number of requests they are processing.
- Therefore each of the Message Processor will allow 100 API requests separately. The net effect is that a total of 200 requests are processed instead of 100 requests.

## Impact

This situation defeats the purpose of the quota configuration and can have detrimental effects on the backend servers that are serving the requests.

The backend servers can:

- Be stressed due to higher than expected incoming traffic
- Become unresponsive to newer API requests leading to 503 errors

## Best Practice

Consider, setting the `<Distributed>` element to `true` in the [Quota policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/quota-policy) to ensure that a common counter is used to track the API requests across all Message Processors. The `<Distributed>` element can be set as shown in the code snippet below:

```xml
<!-- /antipatterns/examples/1-7.xml -->
<Quota name="CheckTrafficQuota">
  <Interval>1</Interval>
  <TimeUnit>hour</TimeUnit>
  <Distributed>true</Distributed>
  <Allow count="100"/>
</Quota>
```
