# Introduction to antipatterns

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/intro

This section is about common antipatterns that arise when API proxies are deployed on Apigee.

## What is an antipattern?

An [*antipattern*](https://en.wikipedia.org/wiki/Anti-pattern) is a software design practice that is ineffective or counterproductive—in other words, the opposite of a "best practice." To put it another way, an antipattern is something that the software allows you to do, but that may have an adverse functional or performance impact.

For example, consider the omnipotent-sounding "God Class/Object". In objected oriented programming, a *god class* is a class that controls too many classes for a given application, as illustrated by the following reference tree:

![Diagram](https://docs.cloud.google.com/apigee/docs/api-platform/images/antipatterns/godclass.png)
 ***Figure 1**: God class*

As the image illustrates, the god class uses and references too many classes.

The framework on which the application was developed does not prevent the creation of such a class, but it has many disadvantages, the primary ones being:

- Hard to maintain
- Single point of failure when the application runs

Consequently, such a class is an antipattern that you should avoid creating.

The good news is that you can identify these antipatterns and rectify them with appropriate best practices, which will make the APIs you deploy on Apigee best serve their intended purpose.

## Summary of antipatterns

The following table lists some common API antipatterns:

| Category | Antipatterns |
| --- | --- |
| Policy antipatterns | • [Issuing refresh tokens without invoking refresh flow](./antipatterns-issuing-refresh-tokens.md) <br>• [Use waitForComplete() in JavaScript code](./antipatterns-wait-for-complete.md) <br>• [Set a long expiration time for OAuth tokens](./antipatterns-oauth-long-expiration.md) <br>• [Use greedy quantifiers in the RegularExpressionProtection policy](./antipatterns-greedy-quantifiers.md) <br>• [Cache error responses](./antipatterns-caching-error.md) <br>• [Store data greater than 256 KB size in cache](./antipatterns-caching-large.md) <br>• [Invoke MessageLogging multiple times in an API proxy](./antipatterns-messagelogging-multiple-times.md) <br>• [High-cardinality identifiers in Quota policy](./antipatterns-high-cardinality-identifiers-quota-policy.md) <br>• [Configure non-distributed quota](./antipatterns-non-distributed-quota.md) <br>• [Reuse a Quota policy](./antipatterns-reusing-quota.md) <br>• [Use the RaiseFault policy under inappropriate conditions](./antipatterns-raise-fault-conditions.md) <br>• [Access multi-value HTTP headers incorrectly in an API Proxy](./antipatterns-multi-value-http-headers.md) <br>• [Use Service Callout to invoke backend service in no target proxy](./antipatterns-service-callout-no-target.md) |
| Generic antipatterns | • [Invoke Management API calls from an API Proxy](./antipatterns-invoking-management.md) <br>• [Invoke a proxy within a proxy using custom code or as a target](./antipatterns-proxy-within-proxy.md) <br>• [Manage Apigee resources without using source control management](./antipatterns-no-source-control.md) <br>• [Load Balance with a single target server with MaxFailures set to a non-zero value](./antipatterns-load-balancing-maxfailures.md) <br>• [Access the request/response payload when streaming is enabled](./antipatterns-payload-with-streaming.md) <br>• [Define multiple ProxyEndpoints in an API Proxy](./antipatterns-multiple-proxyendpoints.md) |
| Backend antipatterns | • [Allow a slow backend](./antipatterns-slow-backend.md) <br>• [Disable HTTP persistent (reusable keep-alive) connections](./antipatterns-disable-persistent-connections.md) |

### Download antipatterns eBook

In addition to the links above, you can also download the antipatterns in eBook format:

- [The Book of Apigee Antipatterns v2.0 (PDF)](https://docs.apigee.com/files/Apigee_Edge_Antipatterns_2_0.pdf)
