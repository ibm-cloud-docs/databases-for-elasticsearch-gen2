---
copyright:
  years: 2026
lastupdated: "2026-08-24"

keywords: elasticsearch, databases, admin user, service credentials, ops manager, elasticsearch managing users, roles, root account

subcollection: databases-for-elasticsearch-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Managing users and roles
{: #user-management}

{{site.data.keyword.databases-for-elasticsearch}} deployments come with authentication enabled and use Elasticsearch's [Built-in users](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/built-in-users.html){: external}.

Add users in the UI using the _Service credentials_ page, with the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-cdb-reference), or the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#createdatabaseuser).

User management procedures for Gen2 VPC architecture have been reviewed and updated for compatibility with VPC-based deployments. Ensure that you follow IAM integration and security best practices.
{: note}

## The `ibm_admin_role` role
{: #user-management-elasticsearch-ibm-superuser}

In {{site.data.keyword.databases-for-elasticsearch}} Gen 2, the `ibm_superuser` role has been replaced with `ibm_admin_role`. All users created through service credentials are automatically assigned the `ibm_admin_role`. You can use these users to create additional users directly through the {{site.data.keyword.databases-for-elasticsearch}} cluster endpoints.

Do not update or replace the `ibm_admin_role` assignment. Changing it disrupts your access to your {{site.data.keyword.databases-for-elasticsearch}} deployment.
{: important}

## Managing users and roles through the UI
{: #user-management-ui}
{: ui}

1. Go to the service dashboard for your service.
2. Click **Service credentials** to open the page.
3. Click **New credential**.
4. Choose a descriptive name for your new credential.
5. Click **Add** to provision the new credentials. A username and password, and an associated Elasticsearch user is auto-generated.

The new credentials appear in the table and the connection strings are available as JSON in a click-to-copy field under _View credentials_.

Creating a user from the CLI or API doesn't automatically populate that user's connection strings into _Service credentials_. If you want to add them there, you can create a new credential with the existing user information. Enter the username and password in the JSON field under _Add Inline Configuration Parameters_. For example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`. Basically, you send in the username and password, and _Service credentials_ generates the connection strings with the credentials completed.

Generating credentials from an existing user does not check for or create that user.
{: .tip}

If you need users that are created from _Service credentials_ to have a different role, use the`admin` user to change their role.

## Managing users and roles through the CLI
{: #user-management-cli}
{: cli}

If you need users to have a different role, you can use the `admin` user to change their role.

Users that are created directly from the CLI do not appear in _Service credentials_, but you can add them.

If you manage your service through the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/cli?topic=cli-install-ibmcloud-cli), create a new user with `cdb user-create`. For example, to create a new user for a deployment named `example-deployment`, use the following command:

```sh
ibmcloud cdb user-create example-deployment <newusername> <newpassword>
```
{: pre}

When the task finishes, retrieve the new user's connection strings with the [`ibmcloud cdb deployment-connections`](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-cdb-reference){: external} command, which looks like:

```sh
ibmcloud cdb deployment-connections [--user <userid>] [--password <password>] [--endpoint-type <endpoint type>] [--all] [--only] [--start] [--certroot <path>] [--json]
```
{: pre}

## Managing users and roles through the API
{: #user-management-api}
{: api}

If you need users to have a different role, use the admin user to change their role.

Users that are created directly from the API do not appear in _Service credentials_, but you can add them.

The _Foundation endpoint_ that is shown on the _Overview_ section of your service provides the base URL to access this deployment through the API. To create and manage users, use the base URL with the [`/users` endpoint](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#createdatabaseuser).

The command looks like:

```sh
curl -X POST 'https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users' \
-H "Authorization: Bearer $APIKEY" \
-H "Content-Type: application/json" \
-d '{"username":"jane_smith", "password":"newsupersecurepassword"}'
```
{: pre}

To retrieve a user's connection strings, use the base URL with the `/users/{userid}/connections` endpoint.

## Elasticsearch-created users and roles
{: #user-management-elasticsearch-users}

If the built-in users and roles do not suit your environment, [create users and roles](https://www.elastic.co/docs/reference/elasticsearch/command-line-tools/users-command){: external} directly in Elasticsearch. The admin user for your deployment has the power to create any role or set of privileges for use on your deployment.
