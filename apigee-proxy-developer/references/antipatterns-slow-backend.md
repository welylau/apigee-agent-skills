# Antipattern: Allow a slow backend

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/slow-backend

Backend systems run the services that API proxies access. In other words, they are the fundamental reason for the very existence of APIs and the API Management Proxy layer.

Any API request that is routed via the Apigee platform traverses a typical path before it hits the backend:

- The request originates from a client which could be anything from a browser to an app.
- The request is then received by the Apigee gateway.
- It is processed within the gateway. As a part of this processing, the request passes onto a number of distributed components.
- The gateway then routes the request to the backend that responds to the request.
- The response from the backend then traverses back the exact reverse path via the Apigee gateway back to the client.

![Diagram](https://docs.cloud.google.com/apigee/docs/api-platform/images/antipatterns/client-edge-backend.png)

In effect, the performance of API requests routed via Apigee is dependent on both Apigee and the backend systems. In this antipattern, we will focus on the impact on API requests due to badly performing backend systems.

## Antipattern

Let us consider the case of a problematic backend. These are the possibilities:

- [Inadequately sized backend](#slow)
- [Slow backend](#inad)

### Inadequately sized backend

The challenge in exposing the services on these backend systems via APIs is that they are accessible to a large number of end users. From a business perspective, this is a desirable challenge, but something that needs to be dealt with.

Many times backend systems are not prepared for this extra demand on their services and are consequently under sized or are not tuned for efficient response.

The problem with an "inadequately sized" backend is that if there is a spike in API requests, then it will stress the resources like CPU, Load and Memory on the backend systems. This would eventually cause API requests to fail.

### Slow backend

The problem with an improperly tuned backend is that it would be very slow to respond to any requests coming to it, thereby leading to increased latencies, premature timeouts and a compromised customer experience.

The Apigee platform offers a few tunable options to circumvent and manage the slow backend. But these options have limitations.

## Impact

- In the case of an inadequately sized backend, increase in traffic could lead to failed requests.
- In the case of a slow backend, the latency of requests will increase.

## Best practice

- Use caching to store the responses to improve the API response times and reduce the load on the backend server.
- Resolve the underlying problem in the slow backend servers.

## Further reading

- [Apigee caching internals](https://docs.cloud.google.com/apigee/docs/api-platform/cache/cache-internals)
