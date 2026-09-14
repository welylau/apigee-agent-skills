# Antipattern: Cache error responses

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/caching-error

Caching is a process of storing data temporarily in a storage area called cache for future reference. Caching data brings significant performance benefits because it:

- Allows faster retrieval of data
- Reduces processing time by avoiding regeneration of data again and again
- Prevents API requests from hitting the backend servers and thereby reduces the overhead on the backend servers
- Allows better utilization of system/application resources
- Improves the response times of APIs

Whenever we have to frequently access some data that doesn't change too often, we highly recommend to use a cache to store this data.

Apigee provides the ability to store data in a cache at runtime for persistence and faster retrieval. The caching feature is made available through the [PopulateCache policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/populate-cache-policy), [LookupCache policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/lookup-cache-policy), [InvalidateCache policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/invalidate-cache-policy), and [ResponseCache policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/response-cache-policy).

In this section, let's look at Response Cache policy. The Response Cache policy in Apigee platform allows you to cache the responses from backend servers. If the client applications make requests to the same backend resource repeatedly and the resource gets updated periodically, then we can cache these responses using this policy. The Response Cache policy helps in returning the cached responses and consequently avoids forwarding the requests to the backend servers unnecessarily.

The Response Cache policy:

- Reduces the number of requests reaching the backend
- Reduces network bandwidth
- Improves API performance and response times

## Antipattern

The [ResponseCache policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/response-cache-policy) lets you cache HTTP responses with any possible Status code, by default. This means that both success and error responses can be cached.

Here's a sample Response Cache policy with default configuration:

```xml
<!-- /antipatterns/examples/1-1.xml -->
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ResponseCache async="false" continueOnError="false" enabled="true" name="TargetServerResponseCache">
  <DisplayName>TargetServer ResponseCache</DisplayName>
  <CacheKey>
    <Key Fragment ref="request.uri" /></CacheKey>
    <Scope>Exclusive</Scope>
    <ExpirySettings>
      <TimeoutInSec ref="flow.variable.here">600</TimeoutInSec>
    </ExpirySettings>
  <CacheResource>targetCache</CacheResource>
</ResponseCache>
```

The Response Cache policy caches error responses in its default configuration. However, it is not advisable to cache error responses without considerable thought on the adverse implications because:

- [Scenario 1](#scenario1): Failures occur for a temporary, unknown period and we may continue to send error responses due to caching even after the problem has been fixed
  **OR**

- [Scenario 2](#scenario2): Failures will be observed for a fixed period of time, then we will have to modify the code to avoid caching responses once the problem is fixed

Let's explain this by taking these two scenarios in more detail.

## Scenario 1: Temporary backend/resource failure

Consider that the failure in the backend server is because of one of the following reasons:

- A temporary network glitch
- The backend server is extremely busy and unable to respond to the requests for a temporary period
- The requested backend resource may be removed/unavailable for a temporary period of time
- The backend server is responding slow due to high processing time for a temporary period, etc

In all these cases, the failures could occur for a unknown time period and then we may start getting successful responses. If we cache the error responses, then we may continue to send error responses to the users even though the problem with the backend server has been fixed.

## Scenario 2: Protracted or fixed backend/resource failure

Consider that we know the failure in the backend is for a fixed period of time. For instance, you are aware that either:

- A specific backend resource will be unavailable for 1 hour
  **OR**

- The backend server is removed/unavailable for 24 hours due to a sudden site failure, scaling issues, maintenance, upgrade, etc.

With this information, we can set the cache expiration time appropriately in the Response Cache policy so that we don't cache the error responses for a longer time. However, once the backend server/resource is available again, we will have to modify the policy to avoid caching error responses. This is because if there is a temporary/one off failure from the backend server, we will cache the response and we will end up with the problem explained in scenario 1 above.

## Impact

- Caching error responses can cause error responses to be sent even after the problem has been resolved in the backend server
- Users may spend a lot of effort troubleshooting the cause of an issue without knowing that it is caused by caching the error responses from the backend server

## Best practice

- Don't store the error responses in the response cache. Ensure that the `<ExcludeErrorResponse>` element is set to `true` in the [ResponseCache policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/response-cache-policy) to prevent error responses from being cached as shown in the below code snippet. With this configuration only the responses for the default success codes 200 to 205 (unless the success codes are modified) will be cached.
  ```xml
  <!-- /antipatterns/examples/1-2.xml -->
  <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
  <ResponseCache async="false" continueOnError="false" enabled="true" name="TargetServerResponseCache">
    <DisplayName>TargetServerResponseCache</DisplayName>
    <CacheKey>
      <KeyFragment ref="request.uri" />
    </CacheKey>
    <Scope>Exclusive</Scope>
    <ExpirySettings>
      <TimeoutinSec ref="flow.variable.here">600</TimeoutinSec>
    </ExpirySettings>
    <CacheResource>targetCache</CacheResource>
    <ExcludeErrorResponse>true</ExcludeErrorResponse>
  </ResponseCache>
  ```

- If you have the requirement to cache the error responses for some specific reason, then you can determine the maximum/exact duration of time for which the failure will be observed (if possible):

  - Set the Expiration time appropriately to ensure that you don't cache the error responses longer than the time for which the failure can be seen.
  - Use the ResponseCache policy to cache the error responses without the `<ExcludeErrorResponse>` element.

  Do this *only if you are absolutely sure that the backend server failure is not for a brief/temporary period*.

- Apigee does not recommend caching 5xx responses from backend servers.

> **Note:** ### NOTE
> After the failure has been fixed, remove/disable the [ResponseCache policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/response-cache-policy). Otherwise, the error responses for temporary backend server failures may get cached.
