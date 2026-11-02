---

copyright:
  years: 2026
lastupdated: "2026-11-02"

keywords: elasticsearch migration, gen1 to gen2, classic infrastructure, vpc, backup and restore, migration hub, terraform, ibmcloud cli, private endpoints, version upgrade, platinum, enterprise, standard, plan transition

subcollection: databases-for-elasticsearch-gen2

content-type: tutorial

---

{{site.data.keyword.attribute-definition-list}}

# Migrating {{site.data.keyword.databases-for-elasticsearch}} from Gen 1 to Gen 2
{: #migrating-elasticsearch-gen1-to-gen2}
{: toc-content-type="tutorial"}

[Gen 2]{: tag-purple}

Use this tutorial to migrate your {{site.data.keyword.databases-for-elasticsearch}} deployment from Gen 1 (Classic) to Gen 2 (VPC) using the Migration hub in the {{site.data.keyword.cloud_notm}} console, the {{site.data.keyword.cloud_notm}} CLI, the {{site.data.keyword.cloud_notm}} API, or Terraform.
{: shortdesc}

The Migration hub guides you through the migration in a wizard, split into two phases. Phase 1 prepares and provisions your Gen 2 target. Phase 2 handles the cutover and closes out the migration. Elasticsearch does not support read replicas or cross-infrastructure replication, so backup and restore is the only available migration path.

## Which plan applies to you
{: #plan-transition}

The Gen 1 plan you are on determines the Gen 2 plan you migrate to. Both plans follow the same migration steps.

| Gen 1 plan | Gen 2 plan | Plan transition |
| --- | --- | --- |
| Platinum | Enterprise | Plan upgrade — no feature loss expected. |
| Enterprise | Standard | Plan change — review the [plan overview](/docs/databases-for-elasticsearch-gen2?topic=databases-for-elasticsearch-gen2-elastic-offerings) before you begin. |
{: caption="Plan transition by source plan" caption-side="bottom"}

## Key considerations before you begin
{: #key-considerations}

Review these requirements before you start the migration. They apply to all interfaces and to both plan transitions.

### Version requirement
{: #version-requirement}

Elasticsearch v8.19 is required on the Gen 1 source before the migration can proceed. Gen 2 launches with v8.19 only, and the backup and restore migration path is validated for same-version restores only.

If your deployment is on an earlier version, you must complete an in-place upgrade to v8.19 on Gen 1 first. This upgrade is a separate service-impacting event, so plan it as a distinct maintenance activity before scheduling the migration.

| Source version on Gen 1 | Action required | Target version on Gen 2 |
| --- | --- | --- |
| 8.19 | None — ready to migrate | 8.19 |
| Earlier than 8.19 | In-place upgrade to v8.19 on Gen 1 required before migrating | 8.19 |
{: caption="Version readiness for Gen 1 to Gen 2 migration" caption-side="bottom"}

For in-place upgrade instructions, see [Upgrading Elasticsearch](/docs/databases-for-elasticsearch?topic=databases-for-elasticsearch-upgrading){: external}.

### Planned downtime
{: #planned-downtime}

The backup and restore path requires a planned maintenance window. The Gen 1 source is set to read-only while the backup is taken. Write traffic is unavailable until your applications are switched to Gen 2. Plan downtime based on your data volume and the time needed to complete the backup, provision, restore, and validate the Gen 2 deployment.

If an in-place upgrade to v8.19 is also required, that upgrade is a separate downtime event. Schedule it ahead of the migration maintenance window.

### Private endpoints
{: #private-endpoints-requirement}

Gen 2 deployments use private endpoints only. Public endpoints are not available on Gen 2. Before you start the migration, confirm that your application infrastructure can reach private service endpoints. If your applications currently connect over public endpoints, plan the network changes before you begin.

## Planning considerations
{: #planning-considerations}

Keep these points in mind when planning your migration:

* Deployments already on v8.19 can complete the migration in a single maintenance window. Deployments on earlier versions require a separate in-place upgrade on Gen 1 first.
* Gen 2 supports private endpoints only. Plan your VPE gateway and any VPN or VSI connectivity before you start the migration.
* Downtime duration depends on data volume. For larger deployments, estimate backup and restore time in a non-production environment first.
* If you are migrating from the Enterprise plan to the Standard plan, complete your feature review before scheduling the maintenance window. See [plan overview](/docs/databases-for-elasticsearch-gen2?topic=databases-for-elasticsearch-gen2-elastic-offerings).
* Test the full procedure in a non-production environment before scheduling a production cutover.

## Before you begin
{: #prereqs}

Confirm the following before you start the migration:

* You have a Gen 1 {{site.data.keyword.databases-for-elasticsearch}} deployment running v8.19 to migrate.
* You have access to provision a Gen 2 deployment in the target account, resource group, and region.
* A maintenance window is approved for the migration activity.
* You have access to the application configuration so that you can update connection details during cutover.
* If migrating from Enterprise to Standard, you have confirmed the Standard plan meets your requirements.

Run through these checks on the source deployment:

* Confirm the deployment is running Elasticsearch v8.19. If not, complete the in-place upgrade first.
* Confirm there are no in-progress operations such as scaling, backups, or upgrades.
* Verify that a recent successful backup exists.
* Check disk utilization and available capacity.
* Identify whether applications connect over public endpoints, private endpoints, or both.

Run the migration against a development or low-traffic instance first to validate the procedure before scheduling a production cutover.
{: tip}

## Migrating using the console
{: #console-migration}
{: ui}

## Backup and restore migration
{: #backup-restore-migration}
{: ui}

The backup and restore path provisions a new Gen 2 deployment from a backup of the Gen 1 source. For a production migration, stopping writes to Gen 1 before taking the backup ensures the restore captures the latest data. For dev or test migrations, you can skip the write-stop step and restore from the most recent automatic backup instead.

* [Backup and restore Phase 1: Prepare and provision](#br-phase-1)
   * [Backup and restore Step 1: Select migration path](#br-select-path)
   * [Backup and restore Step 2: Assess Gen 1](#br-assess-gen1)
   * [Backup and restore Step 3: Backup Gen 1](#br-backup-gen1)
   * [Backup and restore Step 4: Create Gen 2 from backup](#br-provision-gen2)
* [Backup and restore Phase 2: Connect and cut over](#br-phase-2)
   * [Backup and restore Step 5: Set network access](#br-set-network-access)
   * [Backup and restore Step 6: Cutover](#br-cutover)
   * [Backup and restore Step 7: Finish migration](#br-finish-migration)

## Backup and restore Phase 1: Prepare and provision
{: #br-phase-1}
{: ui}

Phase 1 covers the preparation, backup, and provisioning steps required before the Gen 2 instance is ready.

### Backup and restore Step 1: Select migration path
{: #br-select-path}
{: step}
{: ui}

Open the Migration hub in the {{site.data.keyword.cloud_notm}} console. Under **Data services**, select **Eligible data instances** to see the Gen 1 Classic deployments in your account that are ready to migrate. Find your {{site.data.keyword.databases-for-elasticsearch}} instance, click **Open wizard**, then select **Backup and restore** and click **Start**.

### Backup and restore Step 2: Assess Gen 1
{: #br-assess-gen1}
{: step}
{: ui}

After you select the backup and restore path, the wizard runs an automated assessment of your Gen 1 source deployment to confirm it is ready to migrate. The assessment surfaces any required actions and items that need your manual attention before you can proceed.

Before continuing, confirm that the Elasticsearch v8.19 version requirement is met. If the assessment shows an earlier version, complete the in-place upgrade to v8.19 on Gen 1 before returning to the wizard.

The wizard also surfaces recommendations and contextual information about what changes between Gen 1 and Gen 2. Review these before proceeding, as some settings behave differently on Gen 2.

If you are migrating from the Enterprise plan to the Standard plan, use this step to confirm you have completed your plan feature review. The wizard continues regardless of plan transition direction, but this step is a good final checkpoint before you commit to the migration.
{: tip}

After you address any flagged items, **Refresh** to update the assessment results. When the assessment is clear, click **Next: Backup Gen 1** to continue.

### Backup and restore Step 3: Backup Gen 1
{: #br-backup-gen1}
{: step}
{: ui}

This step takes a backup of the Gen 1 source to use as the restore source for the new Gen 2 deployment. The step has two parts: optionally stopping writes, then selecting a backup option.

#### Stop writes to Gen 1 (optional for production, skip for dev/test)
{: #br-stop-writes}
{: ui}

For a production migration, set the Gen 1 (Classic) source to read-only before creating the backup. This ensures the backup captures a consistent final state with no in-flight writes. You can also disconnect all active connections to prevent any new writes during the backup window.

For dev or test migrations, do not set the source to read-only. Setting the source to read-only before you are ready for final cutover can result in data loss on the active instance.
{: important}

After writes are stopped, the wizard shows a **Re-enable writes to Gen 1** option in case you need to roll back before taking the backup.

#### Select a backup option
{: #br-select-backup}
{: ui}

The wizard presents two backup options:

| Option | Description | Best for |
| --- | --- | --- |
| Create on-demand backup | Creates a new backup so the restore uses the latest available data from the Gen 1 source. | Production migrations where data completeness is required. |
| Use last backup | Restores from the most recent completed automatic backup. Changes made after that backup are not included. | Dev and test migrations where speed matters more than data completeness. |
{: caption="Backup options for Gen 1" caption-side="bottom"}

For a production migration, use **Create on-demand backup** to ensure the restore includes the latest data.
{: tip}

### Backup and restore Step 4: Create Gen 2 from backup
{: #br-provision-gen2}
{: step}
{: ui}

This step provisions a new Gen 2 (VPC) instance restored from the Gen 1 backup. The Gen 1 source remains available and continues to be billed while the new instance is provisioned.

The wizard shows a **Preview Elasticsearch on Gen 2** summary with the proposed configuration, covering details, resource allocations, service settings, and backup settings. Fields marked **Changed** highlight differences from the Gen 1 source. The summary also reflects the Gen 2 plan (Enterprise or Standard) based on your source plan.

Click **Edit configurations** to adjust the Gen 2 settings before provisioning begins. The edit panel shows your Gen 1 resource allocation for reference and lets you configure:

* **Location and platform**: Target region. Platform is locked to Gen 2.
* **Service details**: Instance name, resource group, hosting model, and tags.
* **Resource allocation**: Host size (vCPU and RAM), disk and member count.
* **Service configuration**: Database version (8.19) and encryption.

Size the Gen 2 deployment to handle production traffic.

Disk allocation should meet or exceed the Gen 1 source.
{: note}

After you confirm, the wizard starts the restore and a progress bar tracks completion. The **Next: Set network access** button becomes available after the Gen 2 instance is running.

## Backup and restore Phase 2: Connect and cut over
{: #br-phase-2}
{: ui}

Phase 2 covers the steps to configure network access, perform the cutover, and finish the migration.

### Backup and restore Step 5: Set network access
{: #br-set-network-access}
{: step}
{: ui}

Configure private connectivity to the Gen 2 (VPC) instance before cutover. All connection types require a Virtual Private Endpoint (VPE) gateway for private access to the Gen 2 deployment.

Click **Open VPE gateway** to create the gateway from the VPC networking console. **Refresh** after setup to confirm the configuration is detected by the wizard.

The VPE gateway is required. The **Next: Cutover** button is not available until this step is complete.
{: important}

### Backup and restore Step 6: Cutover
{: #br-cutover}
{: step}
{: ui}

With the backup and restore path, the Gen 1 source is in read-only state if you stopped writes in Phase 1. The cutover step focuses on switching your applications to the new Gen 2 deployment.

#### Update application endpoints
{: #br-update-endpoints}
{: ui}

Update your application to use the Gen 2 (VPC) endpoint instead of the Gen 1 (Classic) endpoint. The wizard provides the full connection endpoint string, hostname, port, and SSL mode for easy reference.

Gen 2 uses Let's Encrypt TLS certificates. Update any certificate trust stores or pinned certificate authorities in your application accordingly.
{: note}

#### Update authentication credentials
{: #br-update-credentials}
{: ui}

If your application already uses Gen 2 service credentials, no changes are needed. Otherwise, create a new service credential for the Gen 2 deployment and update your application configuration.

#### Redeploy or restart applications
{: #br-redeploy-apps}
{: ui}

Redeploy or restart your applications to apply the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

### Backup and restore Step 7: Finish migration
{: #br-finish-migration}
{: step}
{: ui}

After your applications are confirmed to be running on Gen 2, click **Next: Finish migration** to complete the workflow.

The Finish migration screen shows a side-by-side summary of both instances, including instance status, active connections, last backup timestamp, and disk usage. Additional metrics for both instances are available through IBM Cloud Monitoring.

After finishing, the Gen 1 deployment stays available. Keep it running through your planned retention period before you decommission it.

There is no automated rollback path after cutover. If you need to revert to Gen 1, provision a new instance from your most recent Gen 1 backup.
{: important}

Continue to [Post-migration tasks](#post-migration) to wrap up.

## Migrating using the CLI
{: #cli-migration}
{: cli}

You can migrate your {{site.data.keyword.databases-for-elasticsearch}} instance from Gen 1 (Classic) to Gen 2 (VPC) by using the {{site.data.keyword.cloud_notm}} CLI.

Before you begin:

- Install the [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cli-getting-started){: external}.
- Install the {{site.data.keyword.databases-for}} plug-in:

   ```bash
   ibmcloud plugin install cloud-databases -f
   ```
   {: pre}

- Log in and target the resource group where the Gen 2 instance will be created:

   ```bash
   ibmcloud login --apikey "$IBMCLOUD_API_KEY"
   ibmcloud target -g <resource-group>
   ```
   {: pre}

Migration to Gen 2 is initially available in the us-east (Washington, DC) region, with additional regions to follow. Ensure that the region you specify is a supported Gen 2 region, such as `us-east`.
{: note}

### Backup and restore migration with the CLI
{: #cli-backup-restore-migration}
{: cli}

The backup and restore path creates a new Gen 2 instance from a backup of the Gen 1 source. All writes to Gen 1 after the backup are not included in the Gen 2 instance. Plan a maintenance window and stop application writes before taking a backup to ensure a consistent migration.

#### Phase 1: Capture backup and provision Gen 2
{: #cli-br-phase-1}
{: cli}

1. **Set environment variables.**

   Set `GEN2_PLAN` to the Gen 2 plan ID that corresponds to your Gen 1 plan. See [Which plan applies to you](#plan-transition).

   ```bash
   export CLASSIC_NAME=<gen1-instance-name>
   export GEN2_NAME=<gen2-instance-name>
   export GEN2_REGION=us-east
   export GEN2_PLAN=<gen2-plan-id>
   ```
   {: pre}

2. **Get the Gen 1 CRN.**

   ```bash
   CLASSIC_CRN="$(ibmcloud resource service-instance "$CLASSIC_NAME" --output json | jq -r '.[0].crn')"
   echo "$CLASSIC_CRN"
   ```
   {: pre}

3. **Verify the Elasticsearch version on Gen 1.**

   Confirm that the source deployment is running Elasticsearch v8.19:

   ```bash
   ibmcloud cdb deployment-about "$CLASSIC_CRN"
   ```
   {: pre}

   If the source is not on v8.19, complete an in-place upgrade on Gen 1 first. See [Version requirement](#version-requirement).

4. **Stop writes to Gen 1 and take an on-demand backup.**

   For production migrations, stop application writes to ensure a consistent restore point, then trigger a backup. For development or test migrations where data completeness is not required, you can skip this step.

   ```bash
   ibmcloud cdb backup-now "$CLASSIC_CRN"
   ```
   {: pre}

5. **Retrieve the backup CRN.**

   List backups and wait until one is completed and restorable:

   ```bash
   ibmcloud cdb backups "$CLASSIC_CRN"
   ```
   {: pre}

   The `restore_backup_id` must be the full coupled backup CRN in the following format:

   ```text
   crn:v1:bluemix:public:databases-for-elasticsearch:<region>:a/<account_id>:<instance_id>:backup:<backup_id>
   ```
   {: screen}

   If the output shows only a backup UUID, compose the full CRN from the Gen 1 CRN:

   ```bash
   BACKUP_UUID=<backup-guid>
   BACKUP_CRN="${CLASSIC_CRN%::}:backup:${BACKUP_UUID}"
   ```
   {: pre}

   Passing only the backup UUID causes provisioning to fail. You must use the full backup CRN.
   {: important}

6. **Prepare the provisioning parameters file.**

   Create a `params.json` file with the following content. The `storage_gb` value is per member, in GB, and must meet or exceed the Gen 1 source data volume:

   ```json
   {
     "dataservices": {
       "elasticsearch": {
         "version": "8.19",
         "members": 3,
         "storage_gb": 20,
         "host_flavor": "bx3d.4x20"
       },
       "restore_backup_id": "<full backup CRN from the previous step>"
     }
   }
   ```
   {: codeblock}

   Or generate it from the variable captured earlier:

   ```bash
   jq --arg crn "$BACKUP_CRN" '.dataservices.restore_backup_id = $crn' params.json.example > params.json
   ```
   {: pre}

   Do not use the flat `{"backup_id": ...}` format that is used for Classic restores. In Gen 2, specify the backup CRN in `dataservices.restore_backup_id`.

7. **Create the Gen 2 instance from the backup.**

   ```bash
   ibmcloud resource service-instance-create "$GEN2_NAME" \
     databases-for-elasticsearch "$GEN2_PLAN" "$GEN2_REGION" \
     -p @params.json
   ```
   {: pre}

8. **Wait until the instance is active.**

   Restore can take several hours depending on data volume. Poll the status until `State` shows `active`:

   ```bash
   ibmcloud resource service-instance "$GEN2_NAME"
   ```
   {: pre}

   Do not delete the Gen 1 instance while the restore is running. Deleting the source also deletes its backups.
   {: important}

#### Phase 2: Configure access, cut over, and finish
{: #cli-br-phase-2}
{: cli}

1. **Configure a VPE gateway for private network access.**

   Gen 2 supports private endpoints only. Before switching applications, create a Virtual Private Endpoint (VPE) gateway from the [{{site.data.keyword.cloud_notm}} VPC networking console](https://cloud.ibm.com/infrastructure/provision/endpointGateway){: external} to connect to the new Gen 2 instance.

2. **Create service credentials.**

   ```bash
   ibmcloud resource service-key-create "${GEN2_NAME}-credentials" \
     --instance-name "$GEN2_NAME" \
     -p '{"role":"Manager"}'
   ```
   {: pre}

   Retrieve the credentials and connection details:

   ```bash
   ibmcloud resource service-key "${GEN2_NAME}-credentials" --output json
   ```
   {: pre}

   Pass `role` as a JSON parameter (`-p '{"role":"Manager"}'`), not as the positional `ROLE_NAME` argument. The `databases-for-elasticsearch` service does not have a `Manager` IAM role, and the CLI fails if `Manager` is passed positionally.
   {: note}

3. **Update application endpoints.**

   Update your application to use the Gen 2 private endpoint from the credentials output instead of the Gen 1 endpoint.

   Gen 2 uses Let's Encrypt TLS certificates. Update any certificate trust stores or pinned certificate authorities in your application accordingly.

4. **Redeploy or restart applications.**

   Restart your applications with the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

5. **Finish and decommission Gen 1.**

   After validating that your applications are running correctly on Gen 2, proceed to [Post-migration tasks](#post-migration). Keep the Gen 1 deployment available through your planned retention period before decommissioning it:

   ```bash
   # ibmcloud resource service-instance-delete "$CLASSIC_NAME"   # only after cutover and retention period
   ```
   {: pre}

   There is no automated rollback path after cutover. If you need to revert to Gen 1, provision a new instance from your most recent Gen 1 backup.
   {: important}

## Migrating using the API
{: #api-migration}
{: api}

You can migrate your {{site.data.keyword.databases-for}} instance from Gen 1 (Classic) to Gen 2 (VPC) by using the [{{site.data.keyword.cloud_notm}} Resource Controller API](https://cloud.ibm.com/apidocs/resource-controller/resource-controller){: external} and the [{{site.data.keyword.databases-for}} API](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api){: external}.

Before you begin, obtain an IAM token:

```bash
IAM_TOKEN=$(curl -s -X POST "https://iam.cloud.ibm.com/identity/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn:ibm:params:oauth:grant-type:apikey&apikey=$IBMCLOUD_API_KEY" \
  | jq -r '.access_token')
```
{: pre}

Migration to Gen 2 is initially available in the `us-east` (Washington, DC) region, with additional regions to follow. Ensure that the region you specify is a supported Gen 2 region, such as `us-east`.
{: note}

### Backup and restore migration with the API
{: #api-backup-restore-migration}
{: api}

The backup and restore path creates a new Gen 2 instance from a backup of the Gen 1 source. All writes to Gen 1 after the backup are not included in the Gen 2 instance. Plan a maintenance window and stop application writes before taking a backup to ensure a consistent migration.

#### Phase 1: Capture backup and provision Gen 2
{: #api-br-phase-1}
{: api}

1. **Get the Gen 1 resource group ID and CRN.**

   Retrieve the resource group ID for the resource group where you plan to create the Gen 2 instance:

   ```bash
   curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_groups?account_id=$ACCOUNT_ID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.resources[] | {name, id}'
   ```
   {: pre}

   Retrieve the Gen 1 instance CRN:

   ```bash
   CLASSIC_CRN=$(curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances?name=$CLASSIC_NAME" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     | jq -r '.resources[0].crn')
   echo "$CLASSIC_CRN"
   ```
   {: pre}

2. **Verify the database version on Gen 1.**

   Confirm that the source deployment is running v8.19 before proceeding. If the source is not on v8.19, complete an in-place upgrade on Gen 1 first. For more information, see [Version requirement](#version-requirement).

   Set `GEN2_REGION` to the target Gen 2 region (for example, `us-east`):

   ```bash
   GEN2_REGION=<target-gen2-region>
   ```
   {: pre}

   Then encode the Gen 1 CRN and verify the version:

   ```bash
   ENCODED_CRN=$(jq -rn --arg v "$CLASSIC_CRN" '$v | @uri')
   curl -s -X GET \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.deployment.version'
   ```
   {: pre}

3. **Stop writes to Gen 1 and take an on-demand backup.**

   For production migrations, stop application writes to ensure a consistent restore point, then trigger a backup. For development or test migrations where data completeness is not required, you can skip this step.

   ```bash
   curl -s -X POST \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}/backups" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" | jq '.task'
   ```
   {: pre}

4. **Retrieve the backup CRN.**

   List backups and wait until one is completed and restorable:

   ```bash
   curl -s -X GET \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}/backups" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.backups[] | {id, status, is_restorable}'
   ```
   {: pre}

   Each backup's `id` is already the full backup CRN. Capture the most recent restorable backup automatically:

   ```bash
   BACKUP_CRN=$(curl -s -X GET \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}/backups" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     | jq -r '[.backups[] | select(.is_restorable == true)] | first | .id')
   echo "$BACKUP_CRN"
   ```
   {: pre}

5. **Provision the Gen 2 instance from the backup.**

   The `resource_plan_id` for Gen 2 is `databases-for-elasticsearch-standard-gen2`. Specify `restore_backup_id` directly in `parameters.dataservices`; do not nest it under the service key. Optional service-specific sizing parameters, including `version`, `host_flavor`, `members`, and `storage_gb`, are specified in `parameters.dataservices.elasticsearch`. If these parameters are not provided, the service broker applies default values.

   The service shortname matches the product: `mysql`, `postgresql`, `redis`, `elasticsearch`, or `mongodb`.
   {: note}

   ```bash
   PROVISION_RESPONSE=$(curl -s -X POST \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d "{
       \"name\": \"$GEN2_NAME\",
       \"target\": \"$GEN2_REGION\",
       \"resource_group\": \"$RESOURCE_GROUP_ID\",
       \"resource_plan_id\": \"databases-for-elasticsearch-standard-gen2\",
       \"parameters\": {
         \"dataservices\": {
           \"restore_backup_id\": \"$BACKUP_CRN\"
         }
       }
     }")
   echo "$PROVISION_RESPONSE" | jq '{name: .name, guid: .guid, state: .state}'
   ```
   {: pre}

   Store the returned GUID for polling:

   ```bash
   GEN2_GUID=$(echo "$PROVISION_RESPONSE" | jq -r '.guid')
   ```
   {: pre}

6. **Wait until the instance is active.**

   Restore can take several hours depending on data volume. Poll the status until `state` shows `active`:

   ```bash
   curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/$GEN2_GUID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '{state: .state, last_operation: .last_operation}'
   ```
   {: pre}

   Do not delete the Gen 1 instance while the restore is running. Deleting the source also deletes its backups.
   {: important}

#### Phase 2: Configure access, cut over, and finish
{: #api-br-phase-2}
{: api}

1. **Configure a VPE gateway for private network access.**

   Gen 2 supports private endpoints only. Before switching applications, create a Virtual Private Endpoint (VPE) gateway from the [{{site.data.keyword.cloud_notm}} VPC networking console](https://cloud.ibm.com/infrastructure/provision/endpointGateway){: external} to connect to the new Gen 2 instance.

2. **Create service credentials.**

   Retrieve the Gen 2 instance CRN from the Resource Controller, then create a service credential with the `Manager` role:

   ```bash
   GEN2_CRN=$(curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/$GEN2_GUID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq -r '.crn')

   curl -s -X POST \
     "https://resource-controller.cloud.ibm.com/v2/resource_keys" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d "{
       \"name\": \"${GEN2_NAME}-credentials\",
       \"source\": \"$GEN2_CRN\",
       \"parameters\": {
         \"role\": \"Manager\"
       }
     }" | jq '.credentials'
   ```
   {: pre}

3. **Update application endpoints.**

   Update your application to use the Gen 2 private endpoint from the credentials output instead of the Gen 1 endpoint.

   Gen 2 uses Let's Encrypt certificates rather than self-signed TLS certificates. Update any certificate validation or pinning in your application accordingly.
   {: note}

4. **Redeploy or restart applications.**

   Restart your applications with the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

5. **Finish and decommission Gen 1.**

   After validating that your applications are running correctly on Gen 2, proceed to [Post-migration tasks](#post-migration). Keep the Gen 1 deployment available through your planned retention period before decommissioning it.

## Migrating using Terraform
{: #migrating-using-terraform}
{: terraform}

You can migrate your {{site.data.keyword.databases-for-elasticsearch}} instances from Gen 1 (Classic) to Gen 2 (VPC) by using Terraform and the {{site.data.keyword.cloud_notm}} provider (`ibm_database` resource).

Terraform migrations use the **Backup and restore** path (planned downtime). Elasticsearch does not support read replicas.

Before you begin:

- [Install the Terraform CLI and the IBM Cloud Provider plug-in](/docs/ibm-cloud-provider-for-terraform?topic=ibm-cloud-provider-for-terraform-setup_cli#tf_installation){: external}.
- Ensure that you have an [IBM Cloud API key](/docs/account?topic=account-userapikey&interface=ui#create_user_key){: external}.

Gen 2 is initially available in the us-east (Washington, DC) region, with additional regions to follow. Ensure that the location parameter in your configurations specifies a supported Gen 2 region, such as us-east.
{: note}

### Backup and restore migration with Terraform
{: #tf-backup-restore-migration}
{: terraform}

The backup and restore path restores a Gen 1 backup onto a newly provisioned Gen 2 instance.

#### Phase 1: Capture backup and restore to Gen 2
{: #tf-br-phase-1}
{: terraform}

1. **Verify and upgrade Gen 1 to Elasticsearch v8.19.**

   Ensure that your Gen 1 source resource definition is updated to Elasticsearch version 8.19 before you restore to Gen 2:

   ```hcl
   resource "ibm_database" "gen1_elasticsearch" {
     name              = "es-gen1-production"
     service           = "databases-for-elasticsearch"
     plan              = "<gen1-plan>"
     location          = "us-east"
     resource_group_id = var.resource_group_id
     version           = "8.19"
     adminpassword     = var.admin_password
   }
   ```
   {: codeblock}

   Apply the upgrade:

   ```bash
   terraform apply -target=ibm_database.gen1_elasticsearch
   ```
   {: pre}

2. **Stop writes and capture an on-demand backup.**

   Stop application writes, then trigger an on-demand backup of the Gen 1 instance:

   ```bash
   ibmcloud cdb deployment-backup-now <gen1-instance-name-or-crn>
   ```
   {: pre}

   Retrieve the backup CRN:

   ```bash
   ibmcloud cdb deployment-backups <gen1-instance-name-or-crn>
   ```
   {: pre}

3. **Provision Gen 2 instance from backup.**

   Specify `backup_id` with the **full backup CRN** in your Gen 2 resource definition. Set `plan` to the Gen 2 plan that corresponds to your Gen 1 plan. See [Which plan applies to you](#plan-transition).

   The `backup_id` parameter requires the full CRN format of the backup:

   `crn:v1:bluemix:public:databases-for-elasticsearch:<region>:a/<account_id>:<instance_id>:backup:<backup_id>`

   Example:

   `crn:v1:bluemix:public:databases-for-elasticsearch:us-east:a/df5f422898b540846cefeb9c5ad0aa66:623ebc19-ad61-47f2-84c5-b5d5b8c7a15a:backup:8d29c132-f351-4067-9b7b-0f307fe0ef84`

   Passing only the backup UUID will cause provisioning to fail.
   {: important}

   ```hcl
   resource "ibm_database" "gen2_from_backup" {
     name              = "es-gen2-restored"
     service           = "databases-for-elasticsearch"
     plan              = var.gen2_plan
     location          = "us-east"
     resource_group_id = var.resource_group_id
     version           = "8.19"

     # Restore from Gen 1 backup CRN (must be the full CRN, not just the UUID)
     backup_id         = var.gen1_backup_crn

     service_endpoints = "private"

     group {
       group_id = "member"

       host_flavor {
         id = "bx3d.4x20"
       }

       disk {
         allocation_mb = 61440
       }
     }

     tags = ["env:production", "restored-from:gen1"]
   }
   ```
   {: codeblock}

   Apply the restore:

   ```bash
   terraform apply -target=ibm_database.gen2_from_backup
   ```
   {: pre}

#### Phase 2: Network access and application cutover
{: #tf-br-phase-2}
{: terraform}

1. **Configure private network access and credentials.**

   Follow the shared steps in [Configuring network access and credentials with Terraform](#tf-network-and-credentials) to configure the VPE gateway for `ibm_database.gen2_from_backup.crn` and generate new service credentials.

2. **Switch application endpoints.**

   Update application connection strings with the new private endpoints and Let's Encrypt TLS certificates, then restart your applications. Confirm that they can read existing data and write new data on Gen 2.

   There is no automated rollback path after cutover. If you need to revert to Gen 1, provision a new instance from your most recent Gen 1 backup.
   {: important}

### Configuring network access and credentials with Terraform
{: #tf-network-and-credentials}
{: terraform}

Because Gen 2 deployments only support private service endpoints and use a different credential model than Gen 1, all Gen 2 deployments require a Virtual Private Endpoint (VPE) gateway and new service credentials regardless of the migration path used.

#### 1. Configure VPE gateway for private network access
{: #tf-vpe-setup}
{: terraform}

Create a Virtual Private Endpoint (VPE) gateway in your VPC for your Gen 2 database instance CRN:

```hcl
resource "ibm_is_virtual_endpoint_gateway" "es_vpe" {
  name           = "vpe-es-gen2"
  target {
    crn           = ibm_database.gen2_from_backup.crn
    resource_type = "provider_cloud_service"
  }
  vpc            = var.vpc_id
  resource_group = var.resource_group_id
}
```
{: codeblock}

Apply the VPE gateway:

```bash
terraform apply -target=ibm_is_virtual_endpoint_gateway.es_vpe
```
{: pre}

#### 2. Generate Gen 2 service credentials
{: #tf-credentials-setup}
{: terraform}

Create service credentials for the Gen 2 instance and export the connection strings:

```hcl
resource "ibm_resource_key" "gen2_service_credentials" {
  name                 = "es-gen2-app-credentials"
  resource_instance_id = ibm_database.gen2_from_backup.id
}

output "gen2_connection_strings" {
  value     = ibm_resource_key.gen2_service_credentials.credentials
  sensitive = true
}
```
{: codeblock}

Apply the credentials:

```bash
terraform apply -target=ibm_resource_key.gen2_service_credentials
```
{: pre}

## Post-migration tasks
{: #post-migration}

After migration is complete, regardless of which interface you used:

* Validate application behavior and key business workflows on the Gen 2 deployment.
* Review performance, logs, and monitoring dashboards.
* Confirm that backup policies and alerting are configured on the Gen 2 deployment.
* Update runbooks, inventory records, and internal documentation to reflect the new Gen 2 endpoints and networking model.
* Delete migration-related service credentials that are no longer needed.
* Keep the Gen 1 deployment available through your planned retention period, as required by your rollback policy.
* Decommission the Gen 1 deployment only after validation and retention requirements are met.
