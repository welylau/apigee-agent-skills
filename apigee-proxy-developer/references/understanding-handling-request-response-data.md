Request and Response Variables
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-handling-request-response-data

Summary:
By default, Apigee passes all request and response data (headers, query parameters, form data, and payloads) unchanged between the client and the backend service. However, developers can use flow variables and policies to inspect, modify, or redirect this data.

Key Concepts:
1. Default Behavior: Transparent pass-through of all request and response data.
2. Request Parsing: Apigee automatically populates flow variables from the incoming request URL:
   - `request.verb`: HTTP method (e.g., GET, POST).
   - `proxy.basepath`: The base path configured for the API proxy.
   - `proxy.pathsuffix`: The specific resource path requested after the base path.
   - `request.querystring`: The raw query string parameters.
3. Data Modification Use Cases:
   - Stripping security credentials before sending requests to the backend.
   - Injecting tracking IDs or analytics data.
   - Routing requests to different target endpoints based on message content.
   - Transforming response payloads (e.g., XML to JSON) before returning them to the client.
4. Common Policies for Data Handling:
   - AssignMessage: Creates or modifies HTTP request or response messages.
   - ExtractVariables: Extracts specific parts of a message (headers, JSON/XML paths) into custom variables.
   - JSONtoXML & XMLtoJSON: Converts message payloads between JSON and XML formats.
   - JavaScript / JavaCallout / PythonScript: Allows custom code to interact with request and response variables via the object model.
