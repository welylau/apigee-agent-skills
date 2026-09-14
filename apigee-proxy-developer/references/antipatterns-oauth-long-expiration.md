# Antipattern: Set a long expiration time for OAuth tokens

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/oauth-long-expiration

Apigee provides a set of tools and policies that allow you to implement [OAuth 2.0 token-based authentication](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/oauth-home) to secure your APIs. OAuth2, described in [IETF RFC 6749](https://www.rfc-editor.org/rfc/rfc6749.html), is the most widely supported open standard for authentication and authorization for APIs. It establishes the *token* as a standard format credential that client applications send to API implementations. The API implementation can verify the token to determine whether the client is authorized to access the API.

 Apigee allows developers to generate access and/or refresh tokens by implementing any one of the four OAuth2 grant types - [client credentials](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/oauth-20-client-credentials-grant-type.html), [password](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/implementing-password-grant-type.html), [implicit](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/access-tokens.html#requestinganaccesstokenimplicitgranttype), and [authorization code](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/oauth-v2-policy-authorization-code-grant-type.html) - using the [OAuthv2 policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/oauthv2-policy). In addition, API developers can use Apigee to implement custom grants, including grants following the Token Exchange pattern, as described in [IETF RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693). Client applications then use access tokens to consume secure APIs. Each access token has its own expiry time, which can be set in the [OAuthv2 policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/oauthv2-policy).

 Apigee can optionally generate and return a refresh token along with the access token with some of the grant types. A client uses a refresh token to obtain a new access token after the original access token is revoked or expires. The expiry time for the refresh token can also be set in the [OAuthv2 policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/oauthv2-policy).

## Antipattern

 Setting a long expiration time for an access token or a refresh token in the [OAuthv2 policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/oauthv2-policy) leads to an expanded window of vulnerability in case of token leakage, which represents a security risk. It also can lead to an accumulation of OAuth tokens in the persistent store, which can result in declining performance over time.

### Example 1

 The following example OAuthV2 policy shows a long expiration time of 10 days for the access tokens:

```xml
<OAuthV2 name="OAuth-GenerateAccessToken">
    <Operation>GenerateAccessToken</Operation>
    <ExpiresIn>864000000</ExpiresIn> <!-- 10 days -->
    <RefreshTokenExpiresIn>864000000</RefreshTokenExpiresIn> <!-- 10 days -->
    <SupportedGrantTypes>
      <GrantType>authorization_code</GrantType>
    </SupportedGrantTypes>
    <GenerateResponse enabled="true"/>
</OAuthV2>
```

In the above example:

- The access token lifetime is set to 10 days.
- The refresh token lifetime is also set to 10 days.

#### Impact

Long-lived access tokens represent a security risk. In the event of token leakage or loss, a short-lived token will naturally expire and become useless, while a long-lived token will continue to grant access to the API for a potentially extended period of time, increasing the window of vulnerability.

An access token should have a short lifetime, probably around 30 minutes or less, and that lifetime should be substantially shorter than the lifetime of the refresh token.

### Example 2

 The following example OAuthV2 policy shows a long expiration time of 200 days for refresh tokens:

```xml
<OAuthV2 name="OAuth-GenerateAccessToken">
  <Operation>GenerateAccessToken</Operation>
  <ExpiresIn>1800000</ExpiresIn> <!-- 30 minutes -->
  <RefreshTokenExpiresIn>17280000000</RefreshTokenExpiresIn> <!-- 200 days -->
  <SupportedGrantTypes>
    <GrantType>authorization_code</GrantType>
  </SupportedGrantTypes>
  <GenerateResponse enabled="true"/>
</OAuthV2>
```

In the above example:

- The access token is set with a reasonable and short expiration time of 30 minutes.
- The refresh token is set with a very long expiration time of 200 days.
- If the traffic to this API is 10 requests/second, then it can generate as many as 864,000 tokens in a day.
- Refresh tokens expire after 200 days; they will accumulate in the data store for the entire lifetime.

#### Impact

Extended refresh token lifetime can potentially lead to performance degradation over time, as a large number of tokens will accumulate in the data store. In Apigee hybrid, excessive token accumulation may also contribute to disk space exhaustion in the persistence layer.

## Best practice

 Use an expiration time for OAuth access and refresh tokens that is appropriate for your specific security requirements, to reduce the window of vulnerability for leaked tokens and avoid token accumulation in the data store. A good starting point for access token lifetime is 30 minutes; for refresh token lifetime, start with 24 hours.

 Set the expiration time for refresh tokens in such a way that it is valid for a multiple of the lifetime of the access tokens. For example, if you set 30 minutes for access token, then set the lifetime of the refresh token to be 24 hours, or 7 days, or whatever is appropriate for the user experience you need to support.

## Further reading

- [OAuth 2.0 framework](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/oauth-home)
- [Secure an API with OAuth](https://docs.cloud.google.com/apigee/docs/api-platform/tutorials/secure-calls-your-api-through-oauth-20-client-credentials)
- [Apigee product limits](https://docs.cloud.google.com/apigee/docs/api-platform/reference/limits)
