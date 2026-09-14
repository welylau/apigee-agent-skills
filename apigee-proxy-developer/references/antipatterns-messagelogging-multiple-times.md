# Antipattern: Invoke the MessageLogging policy multiple times in an API proxy

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/messagelogging-multiple-times

Apigee's [MessageLogging policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/message-logging-policy) lets API Proxy developers log custom messages to [Cloud logging](https://cloud.google.com/logging) or syslog endpoints. Any important information related to the API request such as input parameters, request payload, response code, error messages (if any), and so on, can be logged for later reference or for debugging. While the policy uses a background process to perform the logging, there are caveats to using the policy.

## Antipattern

The [MessageLogging policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/message-logging-policy) provides an efficient way to get more information about an API request and debugging any issues that are encountered with the API request. However, using the same MessageLogging policy more than once or having multiple MessageLogging policies log data in chunks in the same API proxy in flows other than the PostClientFlow may have adverse implications. This is because Apigee opens a connection to an external endpoint for a MessageLogging policy. If the policy uses TLS over TCP, as with syslog endpoints, there is the additional overhead of establishing a TLS connection.

Let's explain this with the help of an example API Proxy.

### API proxy

In the following example, a MessageLogging policy named "LogRequestInfo" is placed in the Request flow, and another MessageLogging policy named "LogResponseInfo" is added to the Response flow. Both are in the ProxyEndpoint PreFlow. The LogRequestInfo policy executes in the background as soon as the API proxy receives the request, and the LogResponseInfo policy executes *after* the proxy has received a response from the target server but *before* the proxy returns the response to the API client. This will consume additional system resources as two TLS connections are potentially being established.

In addition, there's a MessageLogging policy named "LogErrorInfo" that gets executed only if there's an error during API proxy execution.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">...
<FaultRules>
    <FaultRule name="fault-logging">
        <Step>
            <Name>LogErrorInfo</Name>
        </Step>
    </FaultRule>
</FaultRules>
<PreFlow name="PreFlow">
    <Request>
      <Step>
        <Name>LogRequestInfo</Name>
      </Step>
    </Request>
  </PreFlow>
  <PreFlow name="PreFlow">
    <Response>
      <Step>
        <Name>LogResponseInfo</Name>
      </Step>
    </Response>
  </PreFlow>...
</ProxyEndpoint>
```

### Message Logging policy

In the following example policy configurations, the data is being logged to third-party log servers using TLS over TCP. If more than one of these policies is used in the same API proxy, the overhead of establishing and managing TLS connections would occupy additional system memory and CPU cycles, leading to performance issues at scale. Similar negative effects occur if using MessageLogging to log to Cloud logging endpoints.

**LogRequestInfo policy**

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MessageLogging name="LogRequestInfo">
  <Syslog>
    <Message>[3f509b58 tag="{organization.name}.{apiproxy.name}.{environment.name}"] Weather request for WOEID {request.queryparam.w}.</Message>
    <Host>logs-01.loggly.com</Host>
    <Port>6514</Port>
    <Protocol>TCP</Protocol>
    <FormatMessage>true</FormatMessage>
    <SSLInfo>
        <Enabled>true</Enabled>
    </SSLInfo>
  </Syslog>
  <logLevel>INFO</logLevel>
</MessageLogging>
```

**LogResponseInfo policy**

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MessageLogging name="LogResponseInfo">
  <Syslog>
    <Message>[3f509b58 tag="{organization.name}.{apiproxy.name}.{environment.name}"] Status: {response.status.code}, Response {response.content}.</Message>
    <Host>logs-01.loggly.com</Host>
    <Port>6514</Port>
    <Protocol>TCP</Protocol>
    <FormatMessage>true</FormatMessage>
    <SSLInfo>
        <Enabled>true</Enabled>
    </SSLInfo>
  </Syslog>
  <logLevel>INFO</logLevel>
</MessageLogging>
```

**LogErrorInfo policy**

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MessageLogging name="LogErrorInfo">
  <Syslog>
    <Message>[3f509b58 tag="{organization.name}.{apiproxy.name}.{environment.name}"] Fault name: {fault.name}.</Message>
    <Host>logs-01.loggly.com</Host>
    <Port>6514</Port>
    <Protocol>TCP</Protocol>
    <FormatMessage>true</FormatMessage>
    <SSLInfo>
        <Enabled>true</Enabled>
    </SSLInfo>
  </Syslog>
  <logLevel>ERROR</logLevel>
</MessageLogging>
```

## Impact

- Increased networking overhead due to establishing connections to the log endpoints multiple times during API Proxy flow.
- If the syslog server is slow or cannot handle the high volume caused by multiple syslog calls, then it will cause back pressure on the message processor, resulting in slow request processing and potentially high latency or 504 Gateway Timeout errors.
-
  If the MessageLogging policy is placed in flows other than PostClient flow, there's a possibility that the information may not be logged, as the MessageLogging policy will not be executed if any failure occurs before the execution of this policy.

  In the previous [ProxyEndpoint example](#proxyendpoint), the information will not be logged under the following circumstances:

  - If any of the policies placed ahead of the LogRequestInfo policy in the request flow fails.
 or
  - If the target server fails with any error (HTTP 4XX, 5XX). In this situation, when a successful response isn't returned, the LogResponseInfo policy won't get executed.

  In both cases, the LogErrorInfo policy will get executed and log only the error-related information.

## Best practice

- Within the proxy flow, use one or more instances of the [ExtractVariables policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/extract-variables-policy) or the [JavaScript policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/javascript-policy) to set all the flow variables that are to be logged into a context variable; this will make them available for the MessageLogging policy that executes later.
- Use a single MessageLogging policy to log all the required data in the PostClientFlow, which gets executed unconditionally.
- If you are using syslog, use the UDP protocol if guaranteed delivery of messages to the syslog server is not required and TLS/SSL is not mandatory.

The MessageLogging policy was designed to be decoupled from actual API functionality, including error handling. Therefore, invoking it in the PostClientFlow, which is outside of request/response processing, means that it will always log data regardless of whether the API succeeded or not.

Here's an example invoking MessageLogging Policy in PostClientFlow:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>...
<PostClientFlow>
        <Request/>
        <Response>
            <Step>
                <Name>LogInfo</Name>
            </Step>
        </Response>
</PostClientFlow>...
```

Here's an example of a MessageLogging policy, LogInfo, that logs all the data:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MessageLogging name="LogInfo">
  <Syslog>
    <Message>[3f509b58 tag="{organization.name}.{apiproxy.name}.{environment.name}"] Weather request for WOEID {woeid} Status: {weather.response.code}, Response {weather.response}, Fault: {fault.name:None}.</Message>
    <Host>logs-01.loggly.com</Host>
    <Port>6514</Port>
    <Protocol>TCP</Protocol>
    <FormatMessage>true</FormatMessage>
    <SSLInfo>
        <Enabled>true</Enabled>
    </SSLInfo>
  </Syslog>
  <logLevel>INFO</logLevel>
</MessageLogging>
```

Because [response variables](https://docs.cloud.google.com/apigee/docs/api-platform/reference/variables-reference.html#response) are not available in PostClientFlow following an Error Flow, it's important to explicitly set `woeid` and `weather.response*` variables using ExtractVariables or JavaScript policies.

## Further reading

- [JavaScript policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/javascript-policy)
- [ExtractVariables policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/extract-variables-policy)
- Having code execute after proxy processing, including when faults occur, with the [PostClientFlow](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/what-are-flows#postclientflow)
