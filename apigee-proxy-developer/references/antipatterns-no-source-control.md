# Antipattern: Manage resources without using source control management

**URL:** https://docs.cloud.google.com/apigee/docs/api-platform/antipatterns/no-source-control

Apigee provides many different types of resources and each of them serve a different purpose. There are certain resources that can be configured (i.e., created, updated, and/or deleted) only through the Apigee UI, Apigee APIs, or tools that use APIs, and by users with the prerequisite roles and permissions. For example, only org admins belonging to a specific organization can configure these resources. That means, these resources cannot be configured by end users through developer portals, nor by any other means. These resources include:

- API proxies
- Shared flows
- API products
- Caches
- KVMs
- Keystores and truststores
- Virtual hosts
- Target servers
- Resource files

While these resources do have restricted access, if any modifications are made to them even by the authorized users, then the historic data simply gets overwritten with the new data. This is due to the fact that these resources are stored in Apigee only as per their current state. The main exceptions to this rule are API proxies and shared flows.

#### API Proxies and Shared Flows under Revision Control

API proxies and shared flows are managed -- in other words, created, updated and deployed -- through revisions. Revisions are sequentially numbered, which enables you to add new changes and save it as a new revision or revert a change by deploying a previous revision of the API proxy/shared flow. At any point in time, there can be only one revision of an API proxy/shared flow deployed in an environment unless the revisions have a different base path.

Although the API proxies and shared flows are managed through revisions, if any modifications are made to an existing revision, there is no way to roll back since the old changes are simply overwritten.

#### Audits and History

Apigee provides the Audits feature that can be helpful in troubleshooting scenarios. These features enable you to view information like who performed specific operations (create, read, update, delete, deploy, and undeploy) and when the operations were performed on the Apigee resources. However, if any update or delete operations are performed on any of the Apigee resources, the audits cannot provide you the older data.

## Antipattern

#### Managing the Apigee resources (listed above) directly through Apigee UI or APIs without using source control system

There's a misconception that Apigee will be able to restore resources to their previous state following modifications or deletes. However, Apigee does not provide restoration of resources to their previous state. Therefore, it is the user's responsibility to ensure that all the data related to Apigee resources is managed through source control management, so that old data can be restored back quickly in case of accidental deletion or situations where any change needs to be rolled back. This is particularly important for production environments where this data is required for runtime traffic.

Let's explain this with the help of a few examples and the kind of impact that can be caused if the data in not managed through a source control system and is modified/deleted knowingly or unknowingly:

#### Example 1: Deletion or modification of API proxy

When an API proxy is deleted, or a change is deployed on an existing revision, the previous code won't be recoverable. If the API proxy contains Java, JavaScript, or Python code that is not managed in a source control management (SCM) system outside Apigee, a lot of development work and effort could be lost.

#### Example 2: Determination of API proxies using specific virtual hosts

A certificate on a virtual host is expiring and that virtual host needs updating. Identifying which API proxies use that virtual host for testing purposes may be difficult if there are many API proxies. If the API proxies are managed in an SCM system outside Apigee, then it would be easy to search the repository.

#### Example 3: Deletion of keystore/truststore

If a keystore/truststore that is used by a virtual host or target server configuration is deleted, it will not be possible to restore it back unless the configuration details of the keystore/truststore, including certificates and/or private keys, are stored in source control.

## Impact

- If any of the Apigee resources are deleted, then it's not possible to recover the resource and its contents from Apigee.
- API requests may fail with unexpected errors leading to outage until the resource is restored back to its previous state.
- It is difficult to search for inter-dependencies between API proxies and other resources in Apigee.

## Best Practice

- Use any standard SCM coupled with a continuous integration and continuous deployment (CICD) pipeline for managing API proxies and shared flows.
- Use any standard SCM for managing the other Apigee resources, including API products, caches, KVMs, target servers, virtual hosts, and keystores.

  - If there are any existing Apigee resources, then use Apigee APIs to get the configuration details for them as a JSON/XML payload and store them in source control management.
  - Manage any new updates to these resources in source control management.
  - If there's a need to create new Apigee resources or update existing resources, then use the appropriate JSON/XML payload stored in source control management and update the configuration in Apigee using APIs.

* Encrypted KVMs cannot be exported in plain text from the API. It is the user's responsibility to keep a record of what values are put into encrypted KVMs.

## Further reading

-  [Source Control for API Proxy Development](https://discuss.google.dev/t/source-control-for-api-proxy-development/17186)
-  [Guide to implementing CI on Apigee](https://discuss.google.dev/t/continuous-integration-for-api-proxies/17197.html)
-  [Maven Deploy Plugin for API Proxies](https://github.com/apigee/apigee-deploy-maven-plugin)
-  [Maven Config Plugin to manage Resources](https://github.com/apigee/apigee-config-maven-plugin)
-  [Apigee APIs (for automating backups)](https://apidocs.apigee.com/management/apis)
