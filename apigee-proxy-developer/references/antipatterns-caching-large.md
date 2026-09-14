# Antipattern: Store data greater than 256 KB size in cache

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/caching-large

Apigee provides the ability to store data in a cache at runtime for persistence and faster retrieval.

- The data is initially stored in the Message Processor's in-memory cache, referred to as *L1 cache*.
- The L1 cache is limited by the amount of memory reserved for it as a percentage of the JVM memory.
- The cached entries are later persisted in *L2 cache*, which is accessible to all Message Processors. More details can be found in the section below.
- The L2 cache does not have any hard limit on the number of cache entries, however the maximum *size* of the entry that can be cached is restricted to 256 KB. The cache size of 256 KB is the recommended size for optimal performance.

## Antipattern

This particular antipattern talks about the implications of exceeding the current cache size restrictions within Apigee.

When data > 256 KB is cached, the consequences are as follows:

- API requests executed for the first time on each of the Message Processors need to get the data independently from the original source (policy or a target server), as entries > 256 KB are not available in L2 cache.
- Storing larger data (> 256 KB) in L1 cache tends to put more stress on the platform resources. It results in the L1 cache memory being filled up faster and hence lesser space being available for other data. As a consequence, one will not be able to cache the data as aggressively as one would like to.
- Cached entries from the Message Processors will be removed when the limit on the number of entries is reached. This causes the data to be fetched from the original source again on the respective Message Processors.

![Two flow diagrams. One for size<=512KB that shows flows between API Proxy and Message Processors and flows between Message Processors and Persistent Storage L2 Cache. One for size>512KB that shows flows between API Proxy and Message Processors and flows between Message Processors and Data/Response not stored in L2 Cache.](https://docs.cloud.google.com/apigee/docs/api-platform/images/antipatterns/256cache.png)

## Impact

- Data of size > 256 KB will not be stored in L2/persistent cache.
- More frequent calls to the original source (either a policy or a target server) leads to increased latencies for the API requests.

## Best practice

- It is preferred to store data of size < 256 KB in cache to get optimum performance.
- If there's a need to store data > 256 KB, then consider:

  - Using any appropriate database for storing large data
    **OR**

  - Compressing the data

## Further reading

- [Cache internals](https://docs.cloud.google.com/apigee/docs/api-platform/cache/cache-internals)
