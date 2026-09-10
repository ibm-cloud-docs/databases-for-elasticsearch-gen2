---

copyright:
  years: 2026
lastupdated: "2026-08-28"

keywords: elasticsearch, databases, elasticsearch connection strings, elasticsearch connection ibm application, Gen 2

subcollection: databases-for-elasticsearch-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Connecting an {{site.data.keyword.cloud_notm}} application
{: #ibmcloud-app}

[Gen 2]{: tag-purple}

Applications running in {{site.data.keyword.cloud_notm}} can be bound to your {{site.data.keyword.databases-for-elasticsearch_full}} deployment.

## Connecting a Kubernetes Service application
{: #connect-kubernetes}

There are two steps to connecting a Cloud databases deployment to a Kubernetes Service application. First, your deployment needs to be bound to your cluster and its connection strings that are stored in a secret. The second step is configuring your application to use the connection strings.

The sample app in the [Connecting a Kubernetes Service Tutorial](/docs/cloud-databases?topic=cloud-databases-tutorial-k8s-app) provides a sample application that uses Node.js and demonstrates how to bind the sample application to a {{site.data.keyword.databases-for}} deployment.
{: .tip}

Before connecting your Kubernetes Service application to a deployment, make sure that the deployment and cluster are both in the same region and resource group.

### Binding your deployment
{: #bind-deployment}

**Private endpoints** - To use a private endpoint, create a service key that Kubernetes can use when binding to the database.

```sh
ibmcloud resource service-key-create <PRIVATE_KEY> --instance-name <DEPLOYMENT_NAME_OR_CRN> --service-endpoint private
```
{: pre}

The private service endpoint is selected with `--service-endpoint private`. After that, you bind the database to the Kubernetes cluster through the private endpoint with the `cluster service bind` command.

```sh
ibmcloud ks cluster service bind <CLUSTER_NAME> <RESOURCE_GROUP> <DEPLOYMENT_NAME_OR_CRN> --key <PRIVATE_KEY>
```
{: pre}

**Verify** - Verify that the Kubernetes secret was created in your cluster namespace. Running the following command, you get the API key for accessing the instance of your deployment that is provisioned in your account.

```sh
kubectl get secrets --namespace=default
```
{: pre}

For more information, see the [Kubernetes service documentation](/docs/containers?topic=containers-service-binding#bind-services).

### Configuring in your Kubernetes app
{: #configure-kubernetes}

When you bind your application to Kubernetes Service, it creates an environment variable from the cluster's secrets. Your deployment's connection information lives in `BINDING` as a JSON object. Load and parse the JSON object into your application to retrieve the information your application's driver needs to make a connection to the database.

The [Connection strings](/docs/databases-for-elasticsearch-gen2?topic=databases-for-elasticsearch-gen2-connection-strings&interface=ui#connection-string-breakdown) page contains a reference of the JSON fields.

For more information, see the [Kubernetes service documentation](https://cloud.ibm.com/docs/containers?topic=containers-service-binding#reference_secret).
