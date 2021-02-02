theforeman.foreman.basic_setup_rhel7
====================================

This role is an opinionated reuse of other roles in the collection, which creates a basic configuration for everything Katello needs to register and patch existing RHEL7 clients.

That includes uploading a subscription manifest an organization, enabling the rhel-7-server-rpms repository and creating a sync plan for it, optionally syncing it immediately, and creating an activation key <name> to use when registering RHEL7 clients.

The subscription manifest will be retrieved from the specified path on the ansible target host; optionally, it can be fetched first from the RHSM portal using the provided login credentials and manifest UUID. It will be uploaded to the specified organization.

The role enables the rhel-7-server-rpms repository with the 7Server release and x86_64 architecture. The manifest must provide access to this content for the role to work properly.

The role creates a sync plan using any of the sync plan intervals supported by the basic [Sync Plan Role](https://github.com/theforeman/foreman-ansible-modules/blob/develop/roles/sync_plans/README.md).

The role creates an activation key with the provided name. This activation key will register client systems in the "Library" lifecycle environment and "Default Organization View" content view, using the subscription auto-attach feature.

Role Variables
--------------

This role supports the [Common Role Variables](https://github.com/theforeman/foreman-ansible-modules/blob/develop/README.md#common-role-variables).

This role supports the same variables used in the [Manifest Role](https://github.com/theforeman/foreman-ansible-modules/blob/develop/roles/manifest/README.md#role-variables).

It also supports:

`sync_plan_name`: Name of the sync plan to create.

`sync_plan_interval`: 'hourly', 'daily', 'weekly', or 'custom cron'. See the [Sync Plan Role Documentation](https://github.com/theforeman/foreman-ansible-modules/blob/develop/roles/manifest/README.md#role-variables) for more information.

`sync_plan_cron_expression`: Required when using the 'custom cron' `sync_plan_interval`.

`sync_plan_sync_date`: Initial sync date for the sync plan, formatted as 'YYYY-MM-DD HH:MM:SS UTC'.

`activation_key_name`: Name of the activation key to create.

Example Playbooks
-----------------

This example downloads a manifest with the provided UUID from the RHSM portal using the provided credentials and copies it to ~/manifest.zip before uploading it to "Default Organization". It then enables the rhel-7-server-rpms repository and creates a sync_plan which syncs the repository at midnight each day. It creates an activation key "RHEL7_Key" to register existing RHEL7 content hosts.

```yaml
- hosts: localhost
  roles:
    - role: theforeman.foreman.basic_setup_rhel7
      vars:
        server_url: https://foreman.example.com
        username: "admin"
        password: "changeme"
        organization: "Default Organization"
        manifest_download: True
        rhsm_username: "happycustomer"
        rhsm_password: "$ecur3p4$$w0rd"
        manifest_uuid: "01234567-89ab-cdef-0123-456789abcdef"
        manifest_path: "~/manifest.zip"
        sync_plan_name: "Daily RHEL7 Sync"
        sync_plan_interval: daily
        sync_plan_sync_date: 2021-02-02 00:00:00 UTC
        activation_key_name: "RHEL7_Key"
```

This example assumes the manifest has already been downloaded to ~/my_subscription_manifesst.zip on localhost and uploads that manifest to the ACME organization. It enables the rhel-7-server-rpms repository and creates a custom cron sync plan for it. It creates an activation key "RHEL7_Key" to register existing RHEL7 content hosts.

```yaml
- hosts: localhost
  roles:
    - role: theforeman.foreman.basic_setup_rhel7
      vars:
        server_url: https://foreman.example.com
        username: "admin"
        password: "changeme"
        organization: "ACME"
        manifest_download: False
        manifest_path: "~/my_subscription_manifest.zip"
        sync_plan_name: "RHEL7 Sync Plan"
        sync_plan_interval: custom cron
        sync_plan_cron_expression: 0 6 8 * *
        sync_plan_sync_date: 2021-02-02 00:00:00 UTC
        activation_key_name: "RHEL7_Key"
```
