---

copyright:
  years: 2019, 2026
lastupdated: "2026-06-09"

keywords: provision cloud databases, terraform, provisioning parameters, cli, resource controller api, provision elasticsearch

subcollection: databases-for-elasticsearch-gen2-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Provisioning
{: #provisioning}

Provision an {{site.data.keyword.databases-for-elasticsearch_full}} Gen 2 deployment through the [catalog](https://cloud.ibm.com/databases/databases-for-elasticsearch-gen2/create){: external}, the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-cli){: external}, the [{{site.data.keyword.databases-for}} API](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api){: external}, through [Terraform](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs/resources/database){: external}, or through pre-built, open-source, and enterprise-ready [Terraform IBM Modules (TIM)](https://registry.terraform.io/modules/terraform-ibm-modules/icd-elasticsearch/ibm/latest){: external}.

## Provisioning through the {{site.data.keyword.cloud_notm}} console
{: #catalog}
{: ui}

Provision from the console by specifying the following parameters.

### Service details
{: #service_details}
{: ui}

- **Service name** - The name can be any string and is the name that is used on the web and in the CLI to identify the new deployment.
- **Resource group** - If you are organizing your services into [resource groups](/docs/account?topic=account-account_setup), specify the resource group in this field. Otherwise, you can leave it at default. For more information, see [Managing resource groups](/docs/account?topic=account-rgs).
- **Location** - The deployment's public cloud region.

### Hosting model
{: #hosting_model}
{: ui}

{{site.data.keyword.databases-for-elasticsearch}} Gen 2 uses **Isolated Compute**: a secure single-tenant offering for complex, highly-performant enterprise workloads. Each deployment runs on dedicated virtual machines with guaranteed resources.

For more information, see [Gen 2 isolated compute](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-isolated-compute).

### Resource allocation
{: #resource_allocation}
{: ui}

Use the table to choose the machine size (CPU and RAM) for your deployment, and specify the disk size. Gen 2 offers six isolated compute sizes with guaranteed resources.

Verify that the available size selections in the UI match the current offerings. The six standard sizes are: 4x16, 8x32, 8x64, 16x64, 32x128, and 30x240 (CPU x RAM in GB).
{: note}

Specify the disk size depending on your requirements. It can be increased after provisioning but cannot be decreased to prevent data loss.
{: note}

### Service configuration
{: #service_configuration}
{: ui}

- **Database version:** [Set only at deployment]{: tag-red} The deployment version of your database. To ensure optimal performance, run the preferred version. The latest minor version is used automatically. For more information, see [Versioning policy](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-db-version-policy){: external}.
- **Database edition:** [Set only at deployment]{: tag-red} Elasticsearch Enterprise with an Enterprise license is available on Gen 2. Platinum license is not currently supported.
- **Encryption:** [Set only at deployment]{: tag-red} If you use [Key Protect](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-reference-key-protect&interface=ui), an instance and key can be selected to encrypt the deployment's disk. If you do not use your own key, the deployment automatically creates and manages its own disk encryption key.
- **Endpoints:** [Set only at deployment]{: tag-red} Gen 2 deployments support **private endpoints only**. All endpoints are private by default and require VPC connectivity through Virtual Private Endpoints (VPE). For more information, see [Service endpoints](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-reference-service-endpoints).

Gen 2 deployments are provisioned in VPC infrastructure and require VPC connectivity. Ensure you have configured Virtual Private Endpoints (VPE) before attempting to connect to your deployment.
{: important}

After you select the appropriate settings, click **Create** to start the provisioning process.

## Provisioning through the CLI
{: #use-cli}
{: cli}

### Create a service instance through the CLI
{: #create-service-instance-cli}
{: cli}

Before provisioning, follow the instructions provided in the documentation to install the [{{site.data.keyword.cloud_notm}} CLI tool](/docs/cli?topic=cli-install-ibmcloud-cli){: external}.

1. Log in to {{site.data.keyword.cloud_notm}}. If you use a federated user ID, it's important that you switch to a one-time passcode (`ibmcloud login --sso`), or use an API key (`ibmcloud --apikey key or @key_file`) to authenticate. For more information about how to log in by using the CLI, see [General CLI (ibmcloud) commands](/docs/cli?topic=cli-ibmcloud_cli#ibmcloud_login){: external} under `ibmcloud login`.

    ```sh
    ibmcloud login
    ```
    {: pre}

2. Provision your database with the following command:

    ```sh
    ibmcloud resource service-instance-create <INSTANCE_NAME> <SERVICE_NAME> <SERVICE_PLAN_NAME> <LOCATION> <RESOURCE_GROUP> -p '{"members_host_flavor": "<members_host_flavor value>"}' --service-endpoints="<Endpoint>"
    ```
    {: pre}

    For example, to provision a {{site.data.keyword.databases-for-elasticsearch}} Gen 2 instance with isolated compute, set the `"members_host_flavor"` parameter to the desired size. Available hosting sizes and their `members_host_flavor value` parameters are listed in [Table 1](#members-host-flavor-parameter-cli). For example, `{"members_host_flavor": "b3c.4x16.encrypted"}`. Note that the host flavor selection includes CPU and RAM sizes (`b3c.4x16.encrypted` is 4 CPU and 16 RAM).

    ```sh
    ibmcloud resource service-instance-create test-database databases-for-elasticsearch enterprise us-south -p '{"members_host_flavor": "b3c.4x16.encrypted"}' --service-endpoints="private"
    ```
    {: pre}

    The fields in the command are described in the table that follows.

    | Field | Description | Flag |
    |-------|------------|------------|
    | `INSTANCE_NAME` [Required]{: tag-red} | The instance name can be any string and is the name that is used on the web and in the CLI to identify the new deployment. |  |
    | `SERVICE_NAME` [Required]{: tag-red} | Name or ID of the service. For {{site.data.keyword.databases-for-elasticsearch}}, use `databases-for-elasticsearch`. |  |
    | `SERVICE_PLAN_NAME` [Required]{: tag-red} | `enterprise` or `platinum`. Note that `platinum` requires an `isolated` host flavor. |  |
    | `LOCATION` [Required]{: tag-red} | The location where you want to deploy. To retrieve a list of regions, use the `ibmcloud regions` command. |  |
    | `RESOURCE_GROUP` | The Resource group name. The default value is `default`. | -g |
    | `--parameters` | JSON file or JSON string of parameters to create service instance | -p |
    | `members_host_flavor` | To provision an Isolated Compute instance, use `{"members_host_flavor": "<members_host_flavor value>"}`. Select desired CPU and RAM configuration. For more information, see the following table or [Gen 2 isolated compute](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-isolated-compute).| |
    | `--service-endpoints` [Required]{: tag-red} | Gen 2 supports `private` endpoints only. For more information, see [Service endpoints](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-reference-service-endpoints){: external}. |  |
    {: caption="Basic command format fields" caption-side="top"}

    In the CLI, `service-endpoints` is a flag, not a parameter.
    {: note}

### The `members host flavor` parameter
{: #members-host-flavor-parameter-cli}
{: cli}

The `members_host_flavor` parameter defines your Compute sizing. Gen 2 uses Isolated Compute with the following available sizes:

| **Members Host flavor** | **members_host_flavor value** |
|:-------------------------:|:---------------------:|
| 4 CPU x 16 RAM            | `b3c.4x16.encrypted`    |
| 8 CPU x 32 RAM            | `b3c.8x32.encrypted`    |
| 8 CPU x 64 RAM            | `m3c.8x64.encrypted`    |
| 16 CPU x 64 RAM           | `b3c.16x64.encrypted`   |
| 32 CPU x 128 RAM          | `b3c.32x128.encrypted`  |
| 30 CPU x 240 RAM          | `m3c.30x240.encrypted`  |
{: caption="Gen 2 host flavor sizing parameter" caption-side="bottom"}

You will see a response like:

    ```text
    Creating service instance INSTANCE_NAME in resource group default of account    USER...
    OK
    Service instance INSTANCE_NAME was created.
    Name:                INSTANCE_NAME
    ID:                  crn:v1:bluemix:public:databases-for-elasticsearch:us-south:a/   40ddc34a846383BGB5b60e:dd13152c-fe15-4bb6-af94-fde0af5303f4::
    GUID:                dd13152c-fe15-4bb6-af94-fde0af56897
    Location:            LOCATION
    State:               provisioning
    Type:                service_instance
    Sub Type:            Public
    Service Endpoints:   private
    Allow Cleanup:       false
    Locked:              false
    Created at:          2023-06-26T19:42:07Z
    Updated at:          2023-06-26T19:42:07Z
    Last Operation:
                          Status    create in progress
                          Message   Started create instance operation
    ```
    {: codeblock}

    - To check provisioning status, use the following command:

      ```sh
      ibmcloud resource service-instance <INSTANCE_NAME>
      ```
      {: pre}

      When complete, you will see a response like:

      ```text
      Retrieving service instance INSTANCE_NAME in resource group default under account USER's Account as USER...
      OK

      Name:                  INSTANCE_NAME
      ID:                    crn:v1:bluemix:public:databases-for-elasticsearch:us-south:a/40ddc34a953a8c02f109835656860e:dd13152c-fe15-4bb6-af94-fde0af5303f4::
      GUID:                  dd13152c-fe15-4bb6-af94-fde5654765
      Location:              <LOCATION>
      Service Name:          databases-for-elasticsearch
      Service Plan Name:     standard
      Resource Group Name:   default
      State:                 active
      Type:                  service_instance
      Sub Type:              Public
      Locked:                false
      Service Endpoints:     private
      Created at:            2023-06-26T19:42:07Z
      Created by:            USER
      Updated at:            2023-06-26T19:53:25Z
      Last Operation:
                              Status    create succeeded
                              Message   Provisioning elasticsearch with version 7.17 (100%)
      ```
      {: codeblock}

    - Optional: To delete a service instance, run the following command:

      ```sh
      ibmcloud resource service-instance-delete <INSTANCE_NAME_OR_CRN>
      ```
      {: pre}

Autoscaling is not currently available on {{site.data.keyword.databases-for}} Gen 2. Monitor your resources using [{{site.data.keyword.monitoringfull}} integration](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-monitoring), which provides metrics for memory, disk space, and disk I/O utilization. To add resources to your instance, manually scale your deployment.
{: note}

### The `--parameters` parameter
{: #flags-params-service-endpoints}
{: cli}

The `service-instance-create` command supports a `-p` parameter, which allows JSON-formatted parameters to be passed to the provisioning process. For example, you can pass Cloud Resource Names (CRNs) as parameter values, which uniquely identify a resource in the cloud. All parameter names and values are passed as strings.

For example, if a database is being provisioned from a particular backup and the new database deployment needs a total of 12 GB of memory across three members, then the command to provision 4 GBs per member looks like:

```sh
ibmcloud resource service-instance-create databases-for-elasticsearch <INSTANCE_NAME> enterprise us-south \
-p \ '{
  "backup_id": "crn:v1:blue:public:databases-for-elasticsearch:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
  "members_memory_allocation_mb": "4096"
}' --service-endpoints="private"
```
{: pre}

## Provisioning through the Resource Controller API
{: #provision-controller-api}
{: api}

Follow these steps to provision by using the [Resource Controller API](https://cloud.ibm.com/apidocs/resource-controller/resource-controller){: external}.

1. Obtain an [IAM token from your API token](https://cloud.ibm.com/apidocs/resource-controller/resource-controller#authentication){: external}.
2. You need to know the ID of the resource group that you would like to deploy to. This information is available through the [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cli-ibmcloud_commands_resource#ibmcloud_resource_groups).

    Use a command like:

    ```sh
    ibmcloud resource groups
    ```
    {: pre}

3. You need to know the region you want to deploy to.

    To list all of the regions that deployments can be provisioned into from the current region, use the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference){: external}.

    The command looks like:

    ```sh
    ibmcloud cdb regions --json
    ```
    {: pre}

4. A host flavor represents fixed sizes of guaranteed resource allocations. To see which host flavors are available in your region, call the [host flavors capability endpoint](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#capability) like this:

    ```sh
    curl -X POST  https://api.{region}.databases.cloud.ibm.com/v5/ibm/capability/flavors  \
      -H 'Authorization: Bearer <>' \
      -H 'Content-Type: application/json' \
      -d '{
        "deployment": {
          "type": "elasticsearch",
          "location": "us-south"
        }
      }'
    ```
    {: pre}

    This returns:

    ```sh
    {
      "deployment": {
        "type": "elasticsearch",
        "location": "us-south",
        "platform": "classic"
      },
      "capability": {
        "flavors": [
          {
            "id": "b3c.4x16.encrypted",
            "name": "4x16",
            "cpu": {
              "allocation_count": 4
            },
            "memory": {
              "allocation_mb": 16384
            },
            "hosting_size": "xs"
          },
          {
            "id": "b3c.8x32.encrypted",
            "name": "8x32",
            "cpu": {
              "allocation_count": 8
            },
            "memory": {
              "allocation_mb": 32768
            },
            "hosting_size": "s"
          },
          {
            "id": "m3c.8x64.encrypted",
            "name": "8x64",
            "cpu": {
              "allocation_count": 8
            },
            "memory": {
              "allocation_mb": 65536
            },
            "hosting_size": "s+"
          },
          {
            "id": "b3c.16x64.encrypted",
            "name": "16x64",
            "cpu": {
              "allocation_count": 16
            },
            "memory": {
              "allocation_mb": 65536
            },
            "hosting_size": "m"
          },
          {
            "id": "b3c.32x128.encrypted",
            "name": "32x128",
            "cpu": {
              "allocation_count": 32
            },
            "memory": {
              "allocation_mb": 131072
            },
            "hosting_size": "l"
          },
          {
            "id": "m3c.30x240.encrypted",
            "name": "30x240",
            "cpu": {
              "allocation_count": 30
            },
            "memory": {
              "allocation_mb": 245760
            },
            "hosting_size": "xl"
          },
        ]
      }
    }
    ```
    {: pre}

    As shown, the Isolated Compute host flavors available to a {{site.data.keyword.databases-for-elasticsearch}} Gen 2 instance in the `us-south` region are:

    - `b3c.4x16.encrypted`
    - `b3c.8x32.encrypted`
    - `m3c.8x64.encrypted`
    - `b3c.16x64.encrypted`
    - `b3c.32x128.encrypted`
    - `m3c.30x240.encrypted`

    To provision or scale your instance to 4 CPUs and `16384` megabytes or RAM, submit the following command:

    ```sh
    {
      "parameters": {
        "members_host_flavor": "`b3c.4x16.encrypted`"
      }
    }
    ```
    {: pre}

    To scale your instance up to 8 CPUs and `32768` megabytes of RAM, submit the following command:

    ```sh
    {
      "parameters": {
        "members_host_flavor": "b3c.8x32.encrypted"
      }
    }
    ```
    {: pre}

5. Once you have all the information, [provision a new resource instance](https://cloud.ibm.com/apidocs/resource-controller/resource-controller#create-resource-instance){: external} with the {{site.data.keyword.cloud_notm}} Resource Controller.

    ```sh
    curl -X POST \
      https://resource-controller.cloud.ibm.com/v2/resource_instances \
      -H "Authorization: Bearer <TOKEN>" \
      -H 'Content-Type: application/json' \
        -d '{
        "name": "<INSTANCE_NAME>",
        "target": "<targeted-region>",
        "resource_group": "RESOURCE_GROUP_ID",
        "resource_plan_id": "<SERVICE_PLAN_NAME>",
        "parameters": {
            "members_host_flavor": "<members_host_flavor_value>",
            "service_endpoints":" <ENDPOINT>",
            "version":"<VERSION>"
        }
      }'
    ```
    {: pre}

    Provision a {{site.data.keyword.databases-for-elasticsearch}} Gen 2 instance with the `"members_host_flavor"` parameter set to the desired Isolated size. Available hosting sizes and their `members_host_flavor value` parameters are listed in [Table 1](#members_host-flavor-parameter-api). For example, `{"members_host_flavor": "b3c.4x16.encrypted"}`. Note that the host flavor selection includes CPU and RAM sizes (`b3c.4x16.encrypted` is 4 CPU and 16 RAM).

    ```sh
    curl -X POST \
      https://resource-controller.cloud.ibm.com/v2/resource_instances \
      -H "Authorization: Bearer <TOKEN>" \
      -H 'Content-Type: application/json' \
        -d '{ \
        "name": "my-instance", \
        "target": "us-south", \
        "resource_group": "5g9f447903254bb58972a2f3f5a4c711", \
        "resource_plan_id": "<SERVICE_PLAN_NAME>", \
        "parameters": {
        "members_host_flavor": "b3c.4x16.encrypted",
        "service_endpoints": "private"
        } \
      }' \
    ```
    {: pre}

    The parameters `name`, `target`, `resource_group`, `resource_plan_id`, and `service_endpoints` are all required.
    {: required}

    The fields in the command are described in the table that follows.

    | Field | Description | Flag |
    |-------|------------|------------|
    | `NAME` [Required]{: tag-red} | The instance name can be any string and is the name that is used on the web and in the CLI to identify the new deployment. |  |
    | `SERVICE_NAME` [Required]{: tag-red} | Name or ID of the service. For {{site.data.keyword.databases-for-elasticsearch}}, use `databases-for-elasticsearch`. |  |
    | `SERVICE_PLAN_NAME` [Required]{: tag-red} | `enterprise` or `platinum` |  |
    | `TARGET` [Required]{: tag-red} | The region where you want to deploy. To retrieve a list of regions, use the `ibmcloud regions` command. |  |
    | `SERVICE_ENDPOINTS_TYPE` | Gen 2 supports `private` endpoints only. For more information, see [Service endpoints](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-reference-service-endpoints). |  |
    | `RESOURCE_GROUP` | The Resource group name. The default value is `default`. | -g |
    | `--parameters` | JSON file or JSON string of parameters to create service instance | -p |
    | `members_host_flavor` | To provision an Isolated Compute instance, use `{"members_host_flavor": "<members_host_flavor value>"}`. Select desired CPU and RAM configuration. For more information, see the table below, or [Gen 2 isolated compute](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-isolated-compute).| |
    | `service-endpoints` [Required]{: tag-red} | Gen 2 supports `private` endpoints only. For more information, see [Service endpoints](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-reference-service-endpoints){: external}. | |
    {: caption="Basic command format fields" caption-side="top"}

### The `members host flavor` parameter
{: #members_host-flavor-parameter-api}
{: api}

The `members_host_flavor` parameter defines your Compute sizing. Gen 2 uses Isolated Compute with the following available sizes:

| **Members host flavor** | **members_host_flavor value** |
|:-------------------------:|:---------------------:|
| 4 CPU x 16 RAM            | `b3c.4x16.encrypted`    |
| 8 CPU x 32 RAM            | `b3c.8x32.encrypted`    |
| 8 CPU x 64 RAM            | `m3c.8x64.encrypted`    |
| 16 CPU x 64 RAM           | `b3c.16x64.encrypted`   |
| 32 CPU x 128 RAM          | `b3c.32x128.encrypted`  |
| 30 CPU x 240 RAM          | `m3c.30x240.encrypted`  |
{: caption="Gen 2 host flavor sizing parameter" caption-side="bottom"}

Autoscaling is not currently available on {{site.data.keyword.databases-for}} Gen 2. Monitor your resources using [{{site.data.keyword.monitoringfull}} integration](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-monitoring), which provides metrics for memory, disk space, and disk I/O utilization. To add resources to your instance, manually scale your deployment.
{: note}

## List of additional parameters
{: #provisioning-parameters-api}
{: api}

In the `--parameters` object you can provide additional information to create your service instance, including:

- `backup_id` - A CRN of a backup resource to restore from. The backup must be created by a database deployment with the same service ID. The backup is loaded after provisioning and the new deployment starts up that uses that data. A backup CRN is in the format `crn:v1:<...>:backup:<uuid>`. If omitted, the database is provisioned empty.
- `version` - The version of the database to be provisioned. If omitted, the database is created with the most recent major and minor version.
- `disk_encryption_key_crn` - The CRN of a KMS key (for example, [{{site.data.keyword.hscrypto}}](/docs/hs-crypto?topic=hs-crypto-get-started) or [{{site.data.keyword.keymanagementserviceshort}}](/docs/key-protect?topic=key-protect-about)), which is then used for disk encryption. A KMS key CRN is in the format `crn:v1:<...>:key:<id>`.
- `backup_encryption_key_crn` - The CRN of a KMS key (for example, [{{site.data.keyword.hscrypto}}](/docs/hs-crypto?topic=hs-crypto-get-started) or [{{site.data.keyword.keymanagementserviceshort}}](/docs/key-protect?topic=key-protect-about)), which is then used for backup encryption. A KMS key CRN is in the format `crn:v1:<...>:key:<id>`.

    To use a key for your backups, you must first [enable the service-to-service delegation](/docs/cloud-databases?topic=cloud-databases-key-protect&interface=api#key-byok).
    {: note}

- `members_disk_allocation_mb` - Total amount of disk to be allocated to the database. For example, if the value is "30720", and there are three members, then the deployment gets 30 GB of disk total, giving 10 GB of disk per member. If omitted, the default value for the database type is used.

## Provisioning with Terraform
{: #provisioning-terraform}
{: terraform}

Use Terraform to manage your infrastructure through the [`ibm_database` Resource for Terraform](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs/resources/database) supports provisioning {{site.data.keyword.databases-for}} deployments. Alternatively, you can use Terraform IBM Modules to manage your infrastructure through [Terraform IBM Modules for {{site.data.keyword.databases-for-elasticsearch}}](https://registry.terraform.io/modules/terraform-ibm-modules/icd-elasticsearch/ibm/latest){: external}.

### Provisioning with Terraform
{: #provisioning-isolated-computer-terraform}
{: terraform}

Provision a {{site.data.keyword.databases-for-elasticsearch}} Gen 2 instance with the `"host_flavor"` parameter set to the desired Isolated size. Available hosting sizes and their `host_flavor value` parameters are listed in [Table 1](#host-flavor-parameter-terraform). For example, `{"host_flavor": "b3c.4x16.encrypted"}`. Note that the host flavor selection includes CPU and RAM sizes (`b3c.4x16.encrypted` is 4 CPU and 16 RAM).

```terraform
data "ibm_resource_group" "group" {
  name = "<your_group>"
}
resource "ibm_database" "<your_database>" {
  name              = "<your_database_name>"
  plan              = "standard"
  location          = "eu-gb"
  service           = "databases-for-elasticsearch"
  resource_group_id = data.ibm_resource_group.group.id
  service_endpoints = "private"
  tags              = ["tag1", "tag2"]
  adminpassword     = "password12"
  # Pin the database version at provision time.
  # If omitted, the newest available major/minor is selected (per service behavior).
  version = "<VERSION_STRING>"                  # e.g. "8.19" for Elasticsearch preferred major
  group {
    group_id = "member"
    host_flavor {
      id = "b3c.8x32.encrypted"
    }
    disk {
      allocation_mb = 256000
    }
  }
  users {
    name     = "user123"
    password = "password12"
  }
  allowlist {
    address     = "172.168.1.1/32"
    description = "desc"
  }
}
output "ICD Elasticsearch database connection string" {
  value = "http://${ibm_database.test_acc.ibm_database_connection.icd_conn}"
}
```
{: codeblock}

### The `host flavor` parameter
{: #host-flavor-parameter-terraform}
{: terraform}

The `host_flavor` parameter defines your Compute sizing. Gen 2 uses Isolated Compute with the following available sizes:

| **Host flavor** | **host_flavor value** |
|:-------------------------:|:---------------------:|
| 4 CPU x 16 RAM            | `b3c.4x16.encrypted`    |
| 8 CPU x 32 RAM            | `b3c.8x32.encrypted`    |
| 8 CPU x 64 RAM            | `m3c.8x64.encrypted`    |
| 16 CPU x 64 RAM           | `b3c.16x64.encrypted`   |
| 32 CPU x 128 RAM          | `b3c.32x128.encrypted`  |
| 30 CPU x 240 RAM          | `m3c.30x240.encrypted`  |
{: caption="Gen 2 host flavor sizing parameter" caption-side="bottom"}

Autoscaling is not currently available on {{site.data.keyword.databases-for}} Gen 2. Monitor your resources using [{{site.data.keyword.monitoringfull}} integration](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-monitoring), which provides metrics for memory, disk space, and disk I/O utilization. To add resources to your instance, manually scale your deployment.
{: note}
