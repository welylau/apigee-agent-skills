Understanding APIs and API Proxies
URL: https://docs.cloud.google.com/apigee/docs/api-platform/fundamentals/understanding-apis-and-api-proxies

Summary:
API Proxies decouple the app-facing API from backend services, shielding apps from backend changes.

Key Concepts:
1. Endpoints:
   - ProxyEndpoint: Defines how client apps consume your APIs. Configures URL, HTTP/HTTPS, and security policies.
   - TargetEndpoint: Defines how the proxy interacts with backend services. Configures forwarding, security, and response formatting.
2. API Proxy Types:
   - Standard: Includes only standard policies, suitable for lightweight solutions, cannot be in API products.
   - Extensible: Includes at least one extensible policy or flow hook, more functionality.
3. Creation Methods: Apigee UI, XML file import, Apigee REST API, or locally using Apigee in VS Code.
4. Revision: Sequential numbering for managing updates, allowing reverting and promoting between environments.
5. Policy: Modules implementing specific management functions (security, rate-limiting, transformation) without writing code.
