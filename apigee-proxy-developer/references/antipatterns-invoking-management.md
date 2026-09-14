# Antipattern: Invoke Apigee API calls from an API proxy

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/invoking-management

Apigee has a powerful utility called the [Apigee API](https://docs.cloud.google.com/apigee/docs/reference/apis/apigee/rest) which offers services such as:

- Deploying or undeploying API proxies
- Configuring virtual hosts, keystores and truststores, and so on
- Creating, deleting and updating entities such as key value maps (KVMs), API products, developer apps, developers, consumer keys, and so on
- Retrieving information about these entities

These services are made accessible through a component called *Management Server* in the Apigee platform. These services can be invoked easily with the help of simple API calls.

Sometimes we may need to use one or more of these services from API proxies at runtime. This is because the entities such as KVMs, OAuth access tokens, API products, developer apps, developers, consumer keys, and so on contain useful information in the form of key-value pairs, custom attributes or as part of its profile.

For instance, you can store the following information in a KVM to make it more secure and accessible at runtime:

- Back-end target URLs
- Environment properties
- Security credentials of backend or third party systems

Similarly, you may want to get the list of API products or the developer's email address at runtime. This information will be available as part of the developer apps profile.

All this information can be effectively used at runtime to enable dynamic behaviour in policies or custom code within Apigee.

## Antipattern

The Apigee APIs are preferred and useful for administrative tasks and should not be used for performing any runtime logic in API proxies flow. This is because:

- Using Apigee APIs to access information about the entities such as KVMs, OAuth access tokens or for any other purpose from API proxies leads to dependency on Management Servers.
- Management Servers are not a part of Apigee runtime components and therefore, they may not be highly available.
- Management Servers also may not be provisioned within the same network or data center and may therefore introduce network latencies at runtime.
- The entries in the Management Servers are cached for a longer period of time, so you may not be able to see the latest data immediately in the API proxies if you perform writes and reads in a short period of time.
- Increases network hops at runtime.

In the code sample below, the Apigee API call is made via the custom JavaScript code to retrieve the information from the KVM:

```javascript
var response = httpClient.send('https://apigee.googleapis.com/v1/organizations/$ORG/environments/$ENV/keyvaluemaps')
```

If the Management Server is unavailable, then the JavaScript code invoking the Apigee API call fails. This subsequently causes the API request to fail.

## Impact

- Introduces additional dependency on Management Servers during runtime. Any failure in Management Servers will affect the API calls.
- User credentials for Apigee APIs need to be stored either locally or in some secure store such as an encrypted KVM.
- Performance implications owing to invoking the management service over the network.
- May not see the updated values immediately due to longer cache expiration in Management Servers.

## Best practice

There are more effective ways of retrieving information from entities such as KVMs, API products, developer apps, developers, consumer keys, and so on at runtime. Here are a few examples:

- Use a [KeyValueMapOperations policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/key-value-map-operations-policy) to access information from KVMs. Here's sample code that shows how to retrieve information from the KVM:
  ```xml
  <!-- /antipatterns/examples/2-6.xml -->
  <KeyValueMapOperations mapIdentifier="urlMap" async="false"
      continueOnError="false" enabled="true" name="GetURLKVM">
    <DisplayName>GetURLKVM</DisplayName>
    <ExpiryTimeInSecs>86400</ExpiryTimeInSecs>
    <Scope>environment</Scope>
    <Get assignTo="urlHosti" index="2">
      <Key>
        <Parameter>urlHost_1</Parameter>
      </Key>
    </Get>
  </KeyValueMapOperations>
  ```

- To access information about API products, developer apps, developers, consumer keys, and so on in the API proxy, you can do either of the following:

  - If your API Proxy flow has a [VerifyAPIKey policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/verify-api-key-policy), then you can access the information using the flow variables populated as part of this policy. Here is sample code that shows how to retrieve the name and created_by information of a Developer App using JavaScript:
    ```xml
    <!-- /antipatterns/examples/2-7.xml -->
    print("Application Name ", context.getVariable(""verifyapikey. VerifyAPIKey.app.name"));
    print("Created by:", context.getVariable("verifyapikey. VerifyAPIKey.app.created_by"));
    ```

  - If your API Proxy flow doesn't have a [VerifyAPIKey policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/verify-api-key-policy), then you can access the profiles of API products, developer apps, and so on using the `AccessEntity` and `ExtractVariables` policies:

    1. Retrieve the profile of developer app with the [AccessEntity policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/access-entity-policy):
      ```xml
      <!-- /antipatterns/examples/2-8.xml -->
      <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
      <AccessEntity async="false" continueOnError="false" enabled="true" name="GetDeveloperApp">
        <DisplayName>GetDeveloperApp</DisplayName>
        <EntityType value="app"></EntityType>
        <EntityIdentifier ref="developer.app.name" type="appname"/>
        <SecondaryIdentifier ref="developer.id" type="developerid"/>
      </AccessEntity>
      ```

    2. Extract the `appId` from developer app with the [ExtractVariables policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/extract-variables-policy):
      ```xml
      <!-- /antipatterns/examples/2-9.xml -->
      <ExtractVariables name="Extract-Developer App-Info">
        <!--
          The source element points to the variable populated by AccessEntity policy.
          The format is <policy-type>.<policy-name>
          In this case, the variable contains the whole developer profile.
        -->
        <Source>AccessEntity.GetDeveloperApp"</Source>
        <VariablePrefix>developerapp</VariablePrefix>
        <XMLPayload>
          <Variable name="appld" type="string">
            <!-- You parse elements from the developer profile using XPath. -->
            <XPath>/App/AppId</XPath>
          </Variable>
        </XMLPayload>
      </ExtractVariables>
      ```

## Further reading

- [KeyValueMapOperations policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/key-value-map-operations-policy)
- [VerifyAPIKey policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/verify-api-key-policy)
- [AccessEntity policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/access-entity-policy)
