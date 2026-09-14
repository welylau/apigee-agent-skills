# Antipattern: Issuing refresh tokens without invoking refresh flow

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/issuing-refresh-tokens

Refresh tokens are used to obtain new access tokens after the original access token has expired or been revoked. Refresh tokens are optionally issued along with access tokens with some of the grant types.

## Antipattern

 Refresh tokens can be issued either by Apigee or via external resources. However, this is an antipattern if the refresh token is never used via the RefreshAccessToken operation.

## Impact

 Persisting refresh tokens unnecessarily negatively impacts both performance and reliability of the authentication system.

## Best practice

### If the refresh token is never needed

 If refresh tokens are not needed, developers should use the 'client credentials' or 'implicit' grant types when generating new access tokens. These grant types do not issue refresh tokens, which is desirable if the refresh token functionality is not required.

### If the proxy performs only read operation with refresh tokens

 Apigee offers [GetOAuthV2Info](https://cloud.google.com/apigee/docs/api-platform/reference/policies/get-oauth-v2-info-policy#refresh-token) which can be used to retrieve refresh token attributes. Developers should not use this policy to validate refresh tokens. It is an antipattern that the refresh token is never used to exchange for a new access token. Note that Apigee can work with [external access and refresh tokens](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/use-third-party-oauth-system). If the refresh token flow happens outside of Apigee, it's highly recommended to use the RefreshAccessToken operation such that any imported refresh tokens no longer valid are properly removed from the Apigee system.

## Further reading

[Refreshing an access token](https://docs.cloud.google.com/apigee/docs/api-platform/security/oauth/access-tokens#refreshinganaccesstoken)
