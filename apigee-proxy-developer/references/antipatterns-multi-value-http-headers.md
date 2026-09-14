# Antipattern: Access multi-value HTTP headers incorrectly in an API Proxy

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/multi-value-http-headers

The HTTP headers are the name value pairs that allow the client applications and backend services to pass additional information about requests and responses respectively. Some simple examples are:

- Authorization request header passes the user credentials to the server:
  ```text
  Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l
  ```

- The `Content-Type` header indicates the type of the request/response content being sent:
  ```text
  Content-Type: application/json
  ```

The HTTP Headers can have one or more values depending on the [header field definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). A multi-valued header will have comma separated values. Here are a few examples of headers that contain multiple values:

- `Cache-Control: no-cache, no-store, must-revalidate`
- `Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8`
- `X-Forwarded-For: 10.125.5.30, 10.125.9.125`

> **Note:** **Note: **The order of the multiple header values is important and must be preserved. For example with `X-Forwarded-For`, the IP addresses are listed in order of network hops from first to last.

Apigee allows the developers to access headers easily using [flow variables](https://docs.cloud.google.com/apigee/docs/api-platform/reference/variables-reference) in any of the policies or conditional flows. Here are the list of variables that can be used to access a specific request or response header in Apigee:

Flow variables:

- `message.header.`header-name``
- `request.header.`header-name``
- `response.header.`header-name``
- `message.header.`header-name`.`N``
- `request.header.`header-name`.`N``
- `response.header.`header-name`.`N``

Javascript objects:

- `context.proxyRequest.headers.`header-name``
- `context.targetRequest.headers.`header-name``
- `context.proxyResponse.headers.`header-name``
- `context.targetResponse.headers.`header-name``

Here's a sample [AssignMessage policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/assign-message-policy) showing how to read the value of a request header and store it into a variable:

```xml
<AssignMessage continueOnError="false" enabled="true" name="assign-message-default">
  <AssignVariable>
    <Name>reqUserAgent</Name>
    <Ref>request.header.User-Agent</Ref>
  </AssignVariable>
</AssignMessage>
```

## Antipattern

Accessing the values of HTTP headers in Apigee policies in a way that returns only the first value is incorrect and can cause issues if the specific HTTP header has more than one value.

The following sections contain examples of header access.

### Example 1: Read a multi-valued Accept header using JavaScript code

Consider that the `Accept` header has multiple values as shown below:

```text
Accept: text/html, application/xhtml+xml, application/xml
```

Here's the JavaScript code that reads the value from `Accept` header:

```javascript
// Read the values from Accept header
var acceptHeaderValues = context.getVariable("request.header.Accept");
```

The above JavaScript code returns only the first value from the `Accept` header, such as `text/html`.

### Example 2: Read a multi-valued Access-Control-Allow-Headers header in AssignMessage or RaiseFault policy

Consider that the `Access-Control-Allow-Headers` header has multiple values as shown below:

```text
Access-Control-Allow-Headers: content-type, authorization
```

Here's the part of code from AssignMessage or RaiseFault policy setting the `Access-Control-Allow-Headers` header:

```xml
<Set>
  <Headers>
    <Header name="Access-Control-Allow-Headers">{request.header.Access-Control-Request-Headers}</Header>
  </Headers>
</Set>
```

The above code sets the Header `Access-Control-Allow-Headers` with only the first value from the request header `Access-Control-Allow-Headers`, in this example `content-type`.

## Impact

1. In both the examples above, notice that only the first value from multi-valued headers are returned. If these values are subsequently used by another policy in the API Proxy flow or by the backend service to perform some function or logic, then it could lead to an unexpected outcome or result.
2. When request header values are accessed and passed onto the target server, API requests could be processed by the backend incorrectly and hence they may give incorrect results.
3. If the client application is dependent on specific header values from the Apigee response, then it may also process incorrectly and give incorrect results.

## Best Practice

1.
  Reference the `request.header.**header_name**.values.string` form of the flow variable to read all the values of a specific header.

  **Example: Sample fragment that could be used in RaiseFault or AssignMessage to read a multi-value header**

  ```xml
  <Set>
    <Headers>
      <Header name="Inbound-Headers">{request.header.Accept.values.string}</Header>
    </Headers>
  </Set>
  ```

2.
  If you want individual access to each of the distinct values, you can use the appropriate built-in flow variables: `request.header.`header_name`.values.count`, `request.header.`header_name`.`N``, `response.header.`header_name`.values.count`, `response.header.`header_name.N``.

  Then iterate to fetch all the values from a specific header in JavaScript or JavaCallout policies.

  **Example: Sample JavaScript code to read a multi-value header**

  ```javascript
  for (var i = 1; i <=context.getVariable('request.header.Accept.values.count'); i++)
  {
    print(context.getVariable('request.header.Accept.' + i));
  }
  ```

> **Note:** **Note: **The comma is the delimiter between distinct header values. This approach will not split values based on other delimiters like semicolon, etc.

  For example, `application/xml;q=0.9, */*;q=0.8` will appear as two values with the above code. The first value is `application/xml;q=0.9`, and the second will be `*/*;q=0.8`.

  If the header values need to be split using semicolon as a delimiter, then you can use `string.split(";")` within the JavaScript callout to separate the distinct values.

3.
  As an alternative, you can use the `substring()` function available within a [message template](https://docs.cloud.google.com/apigee/docs/api-platform/reference/message-template-intro) on the flow variable `request.header.**header_name**.values` to read all the values of a specific header.

  **Example: Use `substring()` within a message template to read a full multi-value header**

  ```xml
  <Set>
    <Headers>
     <Header name="Inbound-Headers">{substring(request.header.Accept.values,1,-1)}</Header>
    </Headers>
  </Set>
  ```

## Further reading

- [How to handle multi-value headers in Javascript?](https://www.googlecloudcommunity.com/gc/Cloud-Product-Articles/How-to-handle-multi-value-headers-in-Javascript/ta-p/76176)
- [IETF RFC 7230 commentary on using comma-separated lists for multi-valued HTTP header fields](https://www.rfc-editor.org/rfc/rfc7230#section-3.2.2)
- [Request and response variables](https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-handling-request-response-data)
- [Flow variables reference](https://docs.cloud.google.com/apigee/docs/api-platform/reference/variables-reference)
