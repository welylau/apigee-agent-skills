# Antipattern: Accessing the request/response payload when streaming is enabled

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/payload-with-streaming

In Apigee, the default behavior is that HTTP request and response payloads are stored in an in-memory buffer before they are processed by the policies in the API Proxy.

If streaming is enabled, then request and response payloads are streamed without modification to the client app (for responses) and the target endpoint (for requests). Streaming is useful especially if an application accepts or returns large payloads, or if there's an application that returns data in chunks over time.

## Antipattern

Accessing the request/response payload with streaming enabled causes Apigee to go back to the default buffering mode.

![Request to Message Processor Quota Polilcy to Message Processor Extract Variables to Target. Target to Message Processor JSONToXML to Response.](https://docs.cloud.google.com/apigee/docs/api-platform/images/antipatterns/payload-with-streaming.png)
 ***Figure 1**: Accessing request/response payload with streaming enabled*

The illustration above shows that we are trying to extract variables from the request payload and converting the JSON response payload to XML using JSONToXML policy. This will disable the streaming in Apigee.

## Impact

- Streaming will be disabled which can lead to increased latencies in processing the data
- Increase in the heap memory usage or `OutOfMemory` errors can be observed on Message Processors due to use of in-memory buffers especially if we have large request/response payloads

## Best practice

- Don't access the request/response payload when streaming is enabled.

## Further reading

- [Streaming requests and responses](https://docs.cloud.google.com/apigee/docs/api-platform/develop/enabling-streaming)
- [How does Apigee streaming work?](https://www.googlecloudcommunity.com/gc/Apigee/How-does-APIGEE-Edge-Streaming-work/td-p/38511)
- [How to handle streaming data together with normal request/response payload in a single API Proxy](https://www.googlecloudcommunity.com/gc/Apigee/How-to-handle-streaming-data-together-with-normal-request/m-p/49986)
- [Best practices for API proxy design and development](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/best-practices-api-proxy-design-and-development)
