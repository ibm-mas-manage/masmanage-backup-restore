Disaster Recovery
===============================================================================
Backup and restore MAS following a catastrophic OCP failure.

Scope: This process can only restore an environment running MAS Core.  It will be expanded to cover backup restore of applications over time.

- IBM Operator Catalog
- IBM Cloud Pak Foundational Services
- MongoDb
- IBM Suite License Service
- IBM Maximo Application Suite Core Platform
- IBM Manage

Backup
-------------------------------------------------------------------------------

### 1. IBM Operator Catalog
Resides in the `openshift-marketplace` namespace by default:

- We need to backup the version of the `ibm-operator-catalog` installed in the cluster

### 2. IBM Cloud Pak Foundational Services
Resides in the `ibm-common-services` namespace by default:

- Running `mas install` will suffice to restore these services
- If using a custom `ClusterIssuer` for MAS, we need to backup the MAS `ClusterIssuer`.  If using OOTB self-signed certificates, a re-install will generate a new self-signed certificate.

### 3. MongoDb
Resides in `mongodb-ce` namespace by default when using MongoDb Community Edition Operator:

- If using in-cluster MongoDb with MongoDb Community Edition Operator running `mas install` will suffice to create a new MongoDb service
- Run `mongodump` on the MAS database
- Run `mongodump` on the SLS database (usually this is in the same MongoDb cluster as the MAS database)

### 4. IBM SLS
Resides in `ibm-sls` namespace by default

- Backup the license file and ID

### 5. Maximo Application Suite Core Platform
Resides in `mas-{instanceId}-core` namespace

- Backup the `ibm-mas` Subscription and OperatorGroup
- Backup the `superuser-credentials` Secret
- Backup the `ibm-entitlement` Secret
- Backup the `Suite.core.mas.ibm.com` resource
- Backup all `Workspace.core.mas.ibm.com` resources
- Backup all `*.config.mas.ibm.com` resources
- Backup all `*.addons.mas.ibm.com` resources

### 6. Manage
- Backup the `ibm-mas-manage` Subscription and OperatorGroup
- Backup the `ibm-entitlement` Secret
- Backup the `{instanceId}-manage-encryptionsecret` Secret
- Backup the `{instanceId}-manage-encryptionsecret-operator` Secret
- Backup the `{instanceId}-{workspaceId}-jdbccfg-workspace-application-binding` Secret
- Backup the `{workspaceId}-*-bundleproperty` Secrets
- Backup the `{workspaceId}-*-nonsystemproperty` Secrets
- Backup the `{instanceId}-{workspaceId}-internal-maximoproperty-secret` Secret
- Backup JMS server secrets `{workspacedId}-manage-d--sb*--asc--sn` Secrets if JMS Server is installed (Optional)
- Backup the `manageapps.apps.ibm.com` resource
- Backup the `manageworkspaces.apps.mas.ibm.com` resource
- Back up all attachments from the server pod (either all or UI) to an external location such as S3 if you are using Manage Attachments


Restore
-------------------------------------------------------------------------------

### 1. IBM Operator Catalog
- Provide the version of the `ibm-operator-catalog` contained in the backup as the MAS Catalog Version when running `ROLE_NAME=ibm_catalogs ansible-playbook ibm.mas_devops.run_role`

### 2. Cloud Pak Foundational Services
- If using a custom `ClusterIssuer` provide the ClusterIssuer information from the backup when running `ROLE_NAME=ibm_common_services ansible-playbook ibm.mas_devops.run_role`

### 3. MongoDb
- If you need to restore the in-cluster MongoDb run `ROLE_NAME=mongodb ansible-playbook ibm.mas_devops.run_role`
- After the install pipeline completes, run `mongorestore` on the MAS database
- After the install pipeline completes, run `mongorestore` on the SLS database

### 4. SLS
- If you need to restore the in-cluster SLS provide the license file and ID contained in the backup when running `ROLE_NAME=sls ansible-playbook ibm.mas_devops.run_role`

### 5. UDS
- If you need to restore the in-cluster UDS run `ROLE_NAME=sls ansible-playbook ibm.mas_devops.run_role`

### 6. MAS Core
- Create the `ibm-mas` namespace
- Restore the `ibm-mas` OperatorGroup and Subscription, verify the operator installs successfully
- Restore the `superuser-credentials` Secret
- Restore the `ibm-entitlement` Secret
- Restore the `Suite.core.mas.ibm.com` resource
- Restore all `Workspace.core.mas.ibm.com` resources
- Restore all `*.config.mas.ibm.com` resources
    - If you are using a new in-cluster MongoDb instance set up in step 3, replace the system-scope MongoCfg in the backup with the new one generated in step 3
    - If you are using a new in-cluster SLS instance set up in step 4, replace the system-scope SLSCfg in the backup with the new one generated in step 4
    - If you are using a new in-cluster UDS instance set up in step 5, replace the system-scope SLSCfg in the backup with the new one generated in step 5
- Restore all `*.addons.mas.ibm.com` resources

### 7. MAS Manage
- Create the `ibm-mas-manage` namespace
- Restore the `ibm-mas-manage` OperatorGroup and Subscription, verify the operator installs successfully
- Restore the `ibm-entitlement` Secret
- Restore the `{instanceId}-manage-encryptionsecret` Secret
- Restore the `{instanceId}-manage-encryptionsecret-operator`Secret
- Restore the `{instanceId}-{workspaceId}-jdbccfg-workspace-application-binding` Secret
- Restore the `{workspaceId}-*-bundleproperty` Secrets
- Restore the `{workspaceId}-*-nonsystemproperty` Secrets
- Restore the `{instanceId}-{workspaceId}-internal--maximoproperty-secret` Secret
- Restore all the JMS secrets `{workspacedId}-manage-d--sb*--asc--sn` if JMS server is intalled (Optional)
- Restore the `manageapps.apps.ibm.com` resource
- Restore the `manageworkspaces.apps.ibm.com` resource
- Restore attachments from the external location if you are using Manage attachments

Please note that in the event of a disaster, data in transit (i.e. queues/topics) cannot be restored. Therefore, it's important to enable message tracking so that the last processed message can be identified and the integration system can resume sending messages. Additionally, to prevent any data loss, storage replication is recommended.

**Database**: Use the provider's backup and restore process to create backups of and restore your database.

- DB2: https://www.ibm.com/docs/en/db2-warehouse?topic=warehouse-backup-restore-disaster-recovery