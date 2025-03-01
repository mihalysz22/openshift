---

copyright:
  years: 2014, 2022
lastupdated: "2022-03-01"

keywords: openshift, version, upgrade, update

subcollection: openshift

---


{{site.data.keyword.attribute-definition-list}}


# Updating clusters, worker nodes, and cluster components
{: #update}

You can install updates to keep your {{site.data.keyword.openshiftlong}} clusters up-to-date.
{: shortdesc}

You must update your cluster by using the {{site.data.keyword.openshiftlong_notm}} API, CLI, or console tools. You can't update your cluster version from OpenShift Container Platform tools such as the {{site.data.keyword.redhat_openshift_notm}} web console.
{: note}

## Updating the master
{: #master}

Periodically, {{site.data.keyword.redhat_openshift_notm}} releases [major, minor, or patch updates](/docs/openshift?topic=openshift-openshift_versions). Updates can affect the API server version or other components in your master. IBM updates the patch version, but you must update the master major and minor versions.
{: shortdesc}

### About updating the master
{: #master-about}

**How do I know when to update the master?**

You are notified in the {{site.data.keyword.cloud_notm}} console and CLI when updates are available, and can also check the [supported versions](/docs/openshift?topic=openshift-openshift_versions) page.

**Can my worker nodes run a later version than the master?**

Your worker nodes can't run a later `major.minor` Kubernetes version than the master. Additionally, your worker nodes can only be one version behind the master version (`n-1`). First, [update your master](#update_master) to the latest Kubernetes version. Then, [update the worker nodes](#worker_node) in your cluster.

Worker nodes can run later patch versions than the master, such as patch versions that are specific to worker nodes for security updates.

**How are patch updates applied?**

By default, patch updates for the master are applied automatically over the course of several days, so a master patch version might show up as available before it is applied to your master. The update automation also skips clusters that are in an unhealthy state or have operations currently in progress. Occasionally, IBM might disable automatic updates for a specific master fix pack, such as a patch that is only needed if a master is updated from one minor version to another. In any of these cases, you can [check the versions changelog](/docs/openshift?topic=openshift-openshift_changelog) for any potential impact and choose to safely use the `ibmcloud oc cluster master update` [command](/docs/containers?topic=containers-kubernetes-service-cli#cs_cluster_update) yourself without waiting for the update automation to apply.

Unlike the master, you must update your workers for each patch version.


**What happens during the master update?**

Your master is highly available with three replica master pods. The master pods have a rolling update, during which only one pod is unavailable at a time. Two instances are up and running so that you can access and change the cluster during the update. Your worker nodes, apps, and resources continue to run.

**Can I roll back the update?**

No, you can't roll back a cluster to a previous version after the update process takes place. Be sure to use a test cluster and follow the instructions to address potential issues before you update your production master.

**What process can I follow to update the master?**

The following diagram shows the process that you can take to update your master.

![Master update process diagram](/images/update-tree.png){: caption="Figure 1. Updating Kubernetes master process diagram" caption-side="bottom"}
{: #update_master}

### Steps to update the cluster master
{: #master-steps}

Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform access role](/docs/openshift?topic=openshift-users#checking-perms).

To update the {{site.data.keyword.redhat_openshift_notm}} master _major_ or _minor_ version:

1. Review the [{{site.data.keyword.redhat_openshift_notm}} changes](/docs/openshift?topic=openshift-openshift_versions) and make any updates marked _Update before master_.
2. Review any [Kubernetes helpful warnings](https://kubernetes.io/blog/2020/09/03/warnings/){: external}, such as deprecation notices.
3. Check the add-ons and plug-ins that are installed in your cluster for any impact that might be caused by updating the cluster version.

    * **Checking add-ons**
        1. List the add-ons in the cluster.
            ```sh
            ibmcloud oc cluster addon ls --cluster <cluster_name_or_ID>
            ```
            {: pre}

        2. Check the supported {{site.data.keyword.redhat_openshift_notm}} version for each add-on that is installed.
            ```sh
            ibmcloud oc addon-versions
            ```
            {: pre}

        3. If the add-on must be updated to run in the {{site.data.keyword.redhat_openshift_notm}} version that you want to update your cluster to, [update the add-on](/docs/openshift?topic=openshift-managed-addons#updating-managed-add-ons).

    * **Checking plug-ins**
        1. In the [Helm catalog](https://cloud.ibm.com/kubernetes/helm){: external}, find the plug-ins that you installed in your cluster.
        2. From the side menu, expand the **SOURCES & TAR FILE** section.
        3. Download and open the source code.
        4. Check the `README.md` or `RELEASENOTES.md` files for supported versions.
        5. If the plug-in must be updated to run in the {{site.data.keyword.redhat_openshift_notm}} version that you want to update your cluster to, update the plug-in by following the plug-in instructions.

4. Update your API server and associated master components by using the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/login) or running the CLI `ibmcloud oc cluster master update` [command](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_cluster_update).
5. Wait a few minutes, then confirm that the update is complete. Review the API server version on the {{site.data.keyword.cloud_notm}} clusters dashboard or run `ibmcloud oc cluster ls`.
6. Install the version of the [`oc cli`](/docs/containers?topic=containers-cs_cli_install#kubectl) that matches the API server version that runs in the master. [Kubernetes does not support](https://kubernetes.io/releases/version-skew-policy/){: external} `oc` client versions that are two or more versions apart from the server version (n +/- 2).

When the master update is complete, you can update your worker nodes, depending on the type of cluster infrastructure provider that you have.
* [Updating classic worker nodes](#worker_node).
* [Updating VPC worker nodes](#vpc_worker_node).



## Updating classic worker nodes
{: #worker_node}

You notice that an update is available for your worker nodes in a [classic infrastructure](/docs/openshift?topic=openshift-infrastructure_providers) cluster. What does that mean? As security updates and patches are put in place for the API server and other master components, you must be sure that the worker nodes remain in sync. You can make two types of updates: updating only the patch version, or updating the `major.minor` version with the patch version.
{: shortdesc}

![Classic infrastructure provider icon.](images/icon-classic-2.svg) Applies to only classic clusters. Have a VPC cluster? See [Updating VPC worker nodes](#vpc_worker_node) instead.
{: note}

For the latest security patches and fixes, make sure to update your worker nodes to the latest patch as soon as possible after it is available. For more information about the latest updates, review the [Changelog](/docs/containers?topic=containers-changelog).
{: tip}

To update {{site.data.keyword.satelliteshort}} worker nodes, see [Updating hosts that are assigned as worker nodes to Satellite-enabled services](/docs/satellite?topic=satellite-host-update-workers).
{: tip}

* **Patch**: A worker node patch update includes security fixes. You can update the classic worker node to the latest patch by using the `ibmcloud oc worker reload` or `update` commands. Keep in mind that the `update` command also updates the worker node to the same `major.minor` version as the master and latest patch version, if a `major.minor` version update is also available.
* **Major.minor**: A `major.minor` update moves up the Kubernetes version of the worker node to the same version as the master. This type of update often includes changes to the Kubernetes API or other behaviors that you must prepare your cluster for. Remember that your worker nodes can only be one version behind the master version (`n-1`). You can update the classic worker node to the same patch by using the `ibmcloud oc worker update` command.

For more information, see [Update types](/docs/containers?topic=containers-cs_versions#update_types).
{: shortdesc}

**What happens to my apps during an update?**

If you run apps as part of a deployment on worker nodes that you update, the apps are rescheduled onto other worker nodes in the cluster. These worker nodes might be in a different worker pool, or if you have stand-alone worker nodes, apps might be scheduled onto stand-alone worker nodes. To avoid downtime for your app, you must ensure that you have enough capacity in the cluster to carry the workload.

**How can I control how many worker nodes go down at a time during an update or reload?**

If you need all your worker nodes to be up and running, consider [resizing your worker pool](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_worker_pool_resize) or [adding stand-alone worker nodes](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_worker_add) to add more worker nodes. You can remove the additional worker nodes after the update is completed.

In addition, you can create a Kubernetes config map that specifies the maximum number of worker nodes that can be unavailable at a time, such as during an update or reload. Worker nodes are identified by the worker node labels. You can use IBM-provided labels or custom labels that you added to the worker node.

**What if I choose not to define a config map?**

When the config map is not defined, the default is used. By default, a maximum of 20% of all your worker nodes in each cluster can be unavailable during the update process.

### Prerequisites
{: #worker-up-prereqs}

Before you update your classic infrastructure worker nodes, review the prerequisite steps.
{: shortdesc}

Updates to worker nodes can cause downtime for your apps and services. Your worker node machine is reimaged, and data is deleted if not [stored outside the pod](/docs/openshift?topic=openshift-storage_planning#persistent_storage_overview).
{: important}

- [Access your {{site.data.keyword.redhat_openshift_notm}} cluster](/docs/openshift?topic=openshift-access_cluster).
- [Update the master](#master). The worker node version can't be higher than the API server version that runs in your Kubernetes master.
- Make any changes that are marked with _Update after master_ in the [{{site.data.keyword.redhat_openshift_notm}} version preparation guide](/docs/openshift?topic=openshift-openshift_versions).
- If you want to apply a patch update, review the [{{site.data.keyword.redhat_openshift_notm}} version changelog](/docs/openshift?topic=openshift-openshift_changelog).
- Consider [adding more worker nodes](/docs/openshift?topic=openshift-add_workers) so that your cluster has enough capacity to rescheduling your workloads during the update.
- Make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform access role](/docs/openshift?topic=openshift-users#checking-perms).

### Updating classic worker nodes in the CLI with a configmap
{: #worker-up-configmap}

Set up a configmap to perform a rolling update of your classic worker nodes.
{: shortdesc}

1. Complete the [prerequisite steps](#worker-up-prereqs).
2. List available worker nodes and note their private IP address.

    ```sh
    ibmcloud oc worker ls --cluster <cluster_name_or_ID>
    ```
    {: pre}

3. View the labels of a worker node. You can find the worker node labels in the **Labels** section of your CLI output. Every label consists of a `NodeSelectorKey` and a `NodeSelectorValue`.
    ```sh
    oc describe node <private_worker_IP>
    ```
    {: pre}

    Example output

    ```sh
    NAME:               10.184.58.3
    Roles:              <none>
    Labels:             arch=amd64
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    failure-domain.beta.kubernetes.io/region=us-south
                    failure-domain.beta.kubernetes.io/zone=dal12
                    ibm-cloud.kubernetes.io/encrypted-docker-data=true
                    ibm-cloud.kubernetes.io/iaas-provider=softlayer
                    ibm-cloud.kubernetes.io/machine-type=u3c.2x4.encrypted
                    kubernetes.io/hostname=10.123.45.3
                    privateVLAN=2299001
                    publicVLAN=2299012
    Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
    CreationTimestamp:  Tue, 03 Apr 2018 15:26:17 -0400
    Taints:             <none>
    Unschedulable:      false
    ```
    {: screen}

4. Create a config map and define the unavailability rules for your worker nodes. The following example shows four checks, the `zonecheck.json`, `regioncheck.json`, `defaultcheck.json`, and a check template. You can use these example checks to define rules for worker nodes in a specific zone (`zonecheck.json`), region (`regioncheck.json`), or for all worker nodes that don't match any of the checks that you defined in the config map (`defaultcheck.json`). Use the check template to create your own check. For every check, to identify a worker node, you must choose one of the worker node labels that you retrieved in the previous step.  

    For every check, you can set only one value for `NodeSelectorKey` and `NodeSelectorValue`. If you want to set rules for more than one region, zone, or other worker node labels, create a new check. Define up to 10 checks in a config map. If you add more checks, they are ignored.
    {: note}

    Example
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: ibm-cluster-update-configuration
      namespace: kube-system
    data:
      drain_timeout_seconds: "120"
      zonecheck.json: |
        {
          "MaxUnavailablePercentage": 30,
          "NodeSelectorKey": "failure-domain.beta.kubernetes.io/zone",
          "NodeSelectorValue": "dal13"
        }
      regioncheck.json: |
        {
          "MaxUnavailablePercentage": 20,
          "NodeSelectorKey": "failure-domain.beta.kubernetes.io/region",
          "NodeSelectorValue": "us-south"
        }
      defaultcheck.json: |
        {
          "MaxUnavailablePercentage": 20
        }
      <check_name>: |
        {
          "MaxUnavailablePercentage": <value_in_percentage>,
          "NodeSelectorKey": "<node_selector_key>",
          "NodeSelectorValue": "<node_selector_value>"
        }
    ```
    {: codeblock}

    `drain_timeout_seconds`
    :    Optional: The timeout in seconds to wait for the [drain](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/){: external} to complete. Draining a worker node safely removes all existing pods from the worker node and reschedules the pods onto other worker nodes in the cluster. Accepted values are integers in the range 1 - 180. The default value is 30.

    `zonecheck.json` and `regioncheck.json`
    :   Two checks that define a rule for a set of worker nodes that you can identify with the specified `NodeSelectorKey` and `NodeSelectorValue`. The `zonecheck.json` identifies worker nodes based on their zone label, and the `regioncheck.json` uses the region label that is added to every worker node during provisioning. In the example, 30% of all worker nodes that have `dal13` as their zone label and 20% of all the worker nodes in `us-south` can be unavailable during the update.

    `defaultcheck.json`
    :   If you don't create a config map or the map is configured incorrectly, the Kubernetes default is applied. By default, only 20% of the worker nodes in the cluster can be unavailable at a time. You can override the default value by adding the default check to your config map. In the example, every worker node that is not specified in the zone and region checks (`dal13` or `us-south`) can be unavailable during the update. 

    `MaxUnavailablePercentage`
    :   The maximum number of nodes that are allowed to be unavailable for a specified label key and value, which is specified as a percentage. A worker node is unavailable during the deploying, reloading, or provisioning process. The queued worker nodes are blocked from updating if it exceeds any defined maximum unavailable percentages. 

    `NodeSelectorKey`
    :   The label key of the worker node for which you want to set a rule. You can set rules for the default labels that are provided by IBM, as well as on worker node labels that you created. If you want to add a rule for worker nodes that belong to one worker pool, you can use the `ibm-cloud.kubernetes.io/machine-type` label.

    `NodeSelectorValue`
    :   The label value that the worker node must have to be considered for the rule that you define.
        
5. Create the configuration map in your cluster.
    ```sh
    oc apply -f <filepath/configmap.yaml>
    ```
    {: pre}

6. Verify that the config map is created.
    ```sh
    oc get configmap --namespace kube-system
    ```
    {: pre}

7. Update the worker nodes.

    ```sh
    ibmcloud oc worker update --cluster <cluster_name_or_ID> --worker <worker_node1_ID> --worker <worker_node2_ID>
    ```
    {: pre}

8. Optional: Verify the events that are triggered by the config map and any validation errors that occur. The events can be reviewed in the  **Events** section of your CLI output.
    ```sh
    oc describe -n kube-system cm ibm-cluster-update-configuration
    ```
    {: pre}

9. Confirm that the update is complete by reviewing the Kubernetes version of your worker nodes.  
    ```sh
    oc get nodes
    ```
    {: pre}

10. Verify that you don't have duplicate worker nodes. Sometimes, older clusters might list duplicate worker nodes with a **`NotReady`** status after an update. To remove duplicates, see [troubleshooting](/docs/containers?topic=containers-cs_duplicate_nodes).

Next steps:
- Repeat the update process with other worker pools.
- Inform developers who work in the cluster to update their `oc` CLI to the version of the Kubernetes master.
- If the Kubernetes dashboard does not display utilization graphs, [delete the `kube-dashboard` pod](/docs/containers?topic=containers-cs_dashboard_graphs).

### Updating classic worker nodes in the console
{: #worker_up_console}

After you set up the config map for the first time, you can then update worker nodes by using the {{site.data.keyword.cloud_notm}} console.
{: shortdesc}

To update worker nodes from the console:
1. Complete the [prerequisite steps](#worker-up-prereqs) and [set up a config map](#worker_node) to control how your worker nodes are updated.
2. From the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/) menu ![Menu icon](../icons/icon_hamburger.svg "Menu icon"), click **{{site.data.keyword.redhat_openshift_notm}}**.
3. From the **Clusters** page, click your cluster.
4. From the **Worker Nodes** tab, select the checkbox for each worker node that you want to update. An action bar is displayed over the table header row.
5. From the action bar, click **Update**.

If you have Portworx installed in your cluster, you must restart the Portworx pods on updated worker nodes. For more information, see [Portworx limitations](/docs/openshift?topic=openshift-portworx#portworx_limitations).



## Updating VPC worker nodes
{: #vpc_worker_node}

You notice that an update is available for your worker nodes in a [VPC infrastructure cluster](/docs/openshift?topic=openshift-infrastructure_providers). What does that mean? As security updates and patches are put in place for the API server and other master components, you must be sure that the worker nodes remain in sync. You can make two types of updates: updating only the patch version, or updating the `major.minor` version with the patch version.
{: shortdesc}

![VPC infrastructure provider icon.](images/icon-vpc-2.svg)  Applies to only VPC clusters. Have a classic cluster? See [Updating classic worker nodes](#worker_node) instead.
{: note}

If you have Portworx deployed in your cluster, follow the steps to [update VPC worker nodes with Portworx volumes](/docs/containers?topic=containers-portworx#portworx_vpc_up).
{: important}

For the latest security patches and fixes, make sure to update your worker nodes to the latest patch as soon as possible after it is available. For more information about the latest updates, review the [Changelog](/docs/containers?topic=containers-changelog).
{: tip}

To update {{site.data.keyword.satelliteshort}} worker nodes, see [Updating hosts that are assigned as worker nodes to Satellite-enabled services](/docs/satellite?topic=satellite-host-update-workers).
{: tip}

* **Patch**: A worker node patch update includes security fixes. You can update the VPC worker node to the latest patch by using the `ibmcloud oc worker replace` command.
* **Major.minor**: A `major.minor` update moves up the Kubernetes version of the worker node to the same version as the master. This type of update often includes changes to the Kubernetes API or other behaviors that you must prepare your cluster for. Remember that your worker nodes can only be one version behind the master version (`n-1`). You can update the VPC worker node to the same patch by using the `ibmcloud oc worker replace` command with the `--update` flag.

For more information, see [Update types](/docs/containers?topic=containers-cs_versions#update_types).

**What happens to my apps during an update?**

If you run apps as part of a deployment on worker nodes that you update, the apps are rescheduled onto other worker nodes in the cluster. These worker nodes might be in a different worker pool. To avoid downtime for your app, you must ensure that you have enough capacity in the cluster to carry the workload, such as by [resizing your worker pools](/docs/openshift?topic=openshift-add_workers#resize_pool).

**What happens to my worker node during an update?**

Your VPC worker node is replaced by removing the old worker node and provisioning a new worker node that runs at the updated patch or `major.minor` version. The replacement worker node is created in the same zone, same worker pool, and with the same flavor as the deleted worker node. However, the replacement worker node is assigned a new private IP address, and loses any custom labels or taints that you applied to the old worker node (worker pool labels and taints are still applied to the replacement worker node).

**What if I replace multiple worker nodes at the same time?**

If you replace multiple worker nodes at the same time, they are deleted and replaced concurrently, not one by one. Make sure that you have enough capacity in your cluster to reschedule your workloads before you replace worker nodes.

**What if a replacement worker node is not created?**

A replacement worker node is not created if the worker pool does not have [automatic rebalancing enabled](/docs/containers?topic=containers-auto-rebalance-off).

### Prerequisites
{: #vpc_worker_prereqs}

Before you update your VPC infrastructure worker nodes, review the prerequisite steps.
{: shortdesc}

Updates to worker nodes can cause downtime for your apps and services. Your worker node machine is removed, and data is deleted if not [stored outside the pod](/docs/openshift?topic=openshift-storage_planning#persistent_storage_overview).
{: important}

- [Access your {{site.data.keyword.redhat_openshift_notm}} cluster](/docs/openshift?topic=openshift-access_cluster).
- [Update the master](#master). The worker node version can't be higher than the API server version that runs in your Kubernetes master.
- Make any changes that are marked with _Update after master_ in the [{{site.data.keyword.redhat_openshift_notm}} version preparation guide](/docs/openshift?topic=openshift-openshift_versions).
- If you want to apply a patch update, review the [Kubernetes clusters](/docs/openshift?topic=openshift-openshift_changelog) version changelog.
- Make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform access role](/docs/openshift?topic=openshift-users#checking-perms).

### Updating VPC worker nodes in the CLI
{: #vpc_worker_cli}

Before you update your VPC worker nodes, review the prerequisite steps.
{: shortdesc}

1. Complete the [prerequisite steps](#vpc_worker_prereqs).
2. Optional: Add capacity to your cluster by [resizing the worker pool](/docs/openshift?topic=openshift-add_workers#resize_pool). The pods on the worker node can be rescheduled and continue running on the added worker nodes during the update.
3. List the worker nodes in your cluster and note the **ID** and **Primary IP** of the worker node that you want to update.
    ```sh
    ibmcloud oc worker ls --cluster <cluster_name_or_ID>
    ```
    {: pre}

4. Replace the worker node to update either the patch version or the `major.minor` version that matches the master version.
    *  To update the worker node to the same `major.minor` version as the master, include the `--update` flag.
        ```sh
        ibmcloud oc worker replace --cluster <cluster_name_or_ID> --worker <worker_node_ID> --update
        ```
        {: pre}

    *  To update the worker node to the latest patch version at the same `major.minor` version, don't include the `--update` flag.
        ```sh
        ibmcloud oc worker replace --cluster <cluster_name_or_ID> --worker <worker_node_ID>
        ```
        {: pre}

5. Repeat these steps for each worker node that you must update.
6. Optional: After the replaced worker nodes are in a **Ready** status, [resize the worker pool](/docs/openshift?topic=openshift-add_workers#resize_pool) to meet the cluster capacity that you want.

If you are running Portworx in your VPC cluster, you must [manually attach your {{site.data.keyword.block_storage_is_short}} volume to your new worker node.](/docs/openshift?topic=openshift-portworx#portworx_vpc_up)
{: note}

### Updating VPC worker nodes in the console
{: #vpc_worker_ui}

You can update your VPC worker nodes in the console. Before you begin, consider [adding more worker nodes](/docs/openshift?topic=openshift-add_workers) to the cluster to help avoid downtime for your apps.
{: shortdesc}

1. Complete the [prerequisite steps](#vpc_worker_prereqs).
2. From the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/) menu ![Menu icon](../icons/icon_hamburger.svg "Menu icon"), click **{{site.data.keyword.redhat_openshift_notm}}**.
3. From the **Clusters** page, click your cluster.
4. From the **Worker Nodes** tab, select the checkbox for each worker node that you want to update. An action bar is displayed over the table header row.
5. From the action bar, click **Update**.



## Updating flavors (machine types)
{: #machine_type}

You can update the flavors, or machine types, of your worker nodes by adding new worker nodes and removing the old ones. For example, if your cluster has deprecated Ubuntu 16 `x1c` or `x2c` worker node flavors, create Ubuntu 18 worker nodes that use flavors with `x3c` in the names.
{: shortdesc}

Before you begin:
- [Access your {{site.data.keyword.redhat_openshift_notm}} cluster](/docs/openshift?topic=openshift-access_cluster).
- If you store data on your worker node, the data is deleted if not [stored outside the worker node](/docs/openshift?topic=openshift-storage_planning#persistent_storage_overview).
- Make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform access role](/docs/openshift?topic=openshift-users#checking-perms).

To update flavors:

1. List available worker nodes and note their private IP address.
    - **For worker nodes in a worker pool**:
        1. List available worker pools in your cluster.
            ```sh
            ibmcloud oc worker-pool ls --cluster <cluster_name_or_ID>
            ```
            {: pre}

        2. List the worker nodes in the worker pool. Note the **ID** and **Private IP**.
            ```sh
            ibmcloud oc worker ls --cluster <cluster_name_or_ID> --worker-pool <pool_name>
            ```
            {: pre}

        3. Get the details for a worker node. In the output, note the zone and either the private and public VLAN ID for classic clusters or the subnet ID for VPC clusters.
            ```sh
            ibmcloud oc worker get --cluster <cluster_name_or_ID> --worker <worker_ID>
            ```
            {: pre}

    - **Deprecated: For stand-alone worker nodes**:
        1. List available worker nodes. Note the **ID** and **Private IP**.
            ```sh
            ibmcloud oc worker ls --cluster <cluster_name_or_ID>
            ```
            {: pre}

        2. Get the details for a worker node and note the zone, the private VLAN ID, and the public VLAN ID.
            ```sh
            ibmcloud oc worker get --cluster <cluster_name_or_ID> --worker <worker_ID>
            ```
            {: pre}

2. List available flavors in the zone.
    ```sh
    ibmcloud oc flavors --zone <zone>
    ```
    {: pre}

3. Create a worker node with the new machine type.
    - **For worker nodes in a worker pool**:
        1. Create a worker pool with the number of worker nodes that you want to replace.
            * ![Classic infrastructure provider icon.](images/icon-classic-2.svg) Classic clusters:
                ```sh
                ibmcloud oc worker-pool create classic --name <pool_name> --cluster <cluster_name_or_ID> --flavor <flavor> --size-per-zone <number_of_workers_per_zone>
                ```
                {: pre}

            * ![VPC infrastructure provider icon.](images/icon-vpc-2.svg) VPC Generation 2 clusters:
                ```sh
                ibmcloud oc worker-pool create vpc-gen2 --name <name> --cluster <cluster_name_or_ID> --flavor <flavor> --size-per-zone <number_of_worker_nodes> --label <key>=<value>
                ```
                {: pre}

        2. Verify that the worker pool is created.
            ```sh
            ibmcloud oc worker-pool ls --cluster <cluster_name_or_ID>
            ```
            {: pre}

        3. Add the zone to your worker pool that you retrieved earlier. When you add a zone, the worker nodes that are defined in your worker pool are provisioned in the zone and considered for future workload scheduling. If you want to spread your worker nodes across multiple zones, choose a [classic](/docs/openshift?topic=openshift-regions-and-zones#zones-mz) or [VPC](/docs/openshift?topic=openshift-regions-and-zones#zones-vpc) multizone location.
            * ![Classic infrastructure provider icon.](images/icon-classic-2.svg) Classic clusters:
                ```sh
                ibmcloud oc zone add classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
                ```
                {: pre}

            * ![VPC infrastructure provider icon.](images/icon-vpc-2.svg) VPC Generation 2 clusters:
                ```sh
                ibmcloud oc zone add vpc-gen2 --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --subnet-id <vpc_subnet_id>
                ```
                {: pre}

    - **Deprecated: For stand-alone worker nodes**:
        ```sh
        ibmcloud oc worker add --cluster <cluster_name> --flavor <flavor> --workers <number_of_worker_nodes> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
        ```
        {: pre}

4. Wait for the worker nodes to be deployed. When the worker node state changes to **Normal**, the deployment is finished.
    ```sh
    ibmcloud oc worker ls --cluster <cluster_name_or_ID>
    ```
    {: pre}

5. Remove the old worker node. **Note**: If you are removing a flavor that is billed monthly (such as bare metal), you are charged for the entire the month.
    - **For worker nodes in a worker pool**:
        1. Remove the worker pool with the old machine type. Removing a worker pool removes all worker nodes in the pool in all zones. This process might take a few minutes to complete.
            ```sh
            ibmcloud oc worker-pool rm --worker-pool <pool_name> --cluster <cluster_name_or_ID>
            ```
            {: pre}

        2. Verify that the worker pool is removed.
            ```sh
            ibmcloud oc worker-pool ls --cluster <cluster_name_or_ID>
            ```
            {: pre}

    - **Deprecated: For stand-alone worker nodes**:
        ```sh
        ibmcloud oc worker rm --cluster <cluster_name> --worker <worker_node>
        ```
        {: pre}

6. Verify that the worker nodes are removed from your cluster.
    ```sh
    ibmcloud oc worker ls --cluster <cluster_name_or_ID>
    ```
    {: pre}

7. Repeat these steps to update other worker pools or stand-alone worker nodes to different flavors.

## Updating cluster components
{: #components}

Your {{site.data.keyword.openshiftlong_notm}} cluster comes with components, such as Ingress, that are installed automatically when you provision the cluster. By default, these components are updated automatically by IBM. However, you can disable automatic updates for some components and manually update them separately from the master and worker nodes.
{: shortdesc}

**What default components can I update separately from the cluster?**

You can optionally disable automatic updates for the following components:
* [Fluentd for logging](#logging-up)
* [Ingress application load balancer (ALB)](#alb)

**Are there components that I can't update separately from the cluster?**

Yes. Your cluster is deployed with the following managed components and associated resources that can't be changed, except to scale pods or edit configmaps for certain performance benefits. If you try to change one of these deployment components, their original settings are restored on a regular interval when they are updated with the cluster master. However, note that resources that you create that are associated with these components, such as Calico network policies that you create to be implemented by the Calico deployment components, are not updated.

* `calico` components
* `coredns` components
* `ibm-cloud-provider-ip`
* `ibm-file-plugin`
* `ibm-keepalived-watcher`
* `ibm-master-proxy`
* `ibm-storage-watcher`
* `kubernetes-dashboard` components
* `metrics-server`
* `olm-operator` and `catalog` components (1.16 and later)
* `vpn`

**Can I install other plug-ins or add-ons than the default components?**

Yes. {{site.data.keyword.openshiftlong_notm}} provides other plugin-ins and add-ons that you can choose from to add capabilities to your cluster. For example, you might want to [use Helm charts](/docs/openshift?topic=openshift-helm) to install  [strongSwan VPN](/docs/openshift?topic=openshift-vpn#vpn-setup). Or you might want to enable IBM-managed add-ons in your cluster, such as the Diagnostics and Debug Tool. You must update these Helm charts and add-ons separately by following the instructions in the Helm chart readme files or by following the steps to [update managed add-ons](/docs/containers?topic=containers-managed-addons#updating-managed-add-ons).

### Managing automatic updates for Fluentd
{: #logging-up}

When you create a logging configuration for a source in your cluster to forward to an external server, a Fluentd component is created in your cluster. To change your logging or filter configurations, the Fluentd component must be at the latest version. By default, automatic updates to the component are enabled.
{: shortdesc}

As of 14 November 2019, a Fluentd component is created for your cluster only if you [create a logging configuration to forward logs to a syslog server](/docs/containers?topic=containers-health#configuring). If no logging configurations for syslog exist in your cluster, the Fluentd component is removed automatically. If you don't forward logs to syslog and want to ensure that the Fluentd component is removed from your cluster, automatic updates to Fluentd must be enabled.
{: important}

You can manage automatic updates of the Fluentd component in the following ways. **Note**: To run the following commands, you must have the [**Administrator** {{site.data.keyword.cloud_notm}} IAM platform access role](/docs/openshift?topic=openshift-users#checking-perms) for the cluster.

* Check whether automatic updates are enabled by running the `ibmcloud oc logging autoupdate get --cluster <cluster_name_or_ID>` [command](/docs/containers?topic=containers-kubernetes-service-cli#cs_log_autoupdate_get).
* Disable automatic updates by running the `ibmcloud oc logging autoupdate disable` [command](/docs/containers?topic=containers-kubernetes-service-cli#cs_log_autoupdate_disable).
* If automatic updates are disabled, but you need to change your configuration, you have two options:
    * Turn on automatic updates for your Fluentd pods.
        ```sh
        ibmcloud oc logging autoupdate enable --cluster <cluster_name_or_ID>
        ```
        {: pre}

    * Force a one-time update when you use a logging command that includes the `--force-update` option. **Note**: Your pods update to the latest version of the Fluentd component, but Fluentd does not update automatically going forward.
        Example command

        ```sh
        ibmcloud oc logging config update --cluster <cluster_name_or_ID> --id <log_config_ID> --type <log_type> --force-update
        ```
        {: pre}

### Managing automatic updates for Ingress ALBs
{: #alb}

Control when the Ingress application load balancer (ALB) component is updated. For information about keeping ALBs up-to-date, see [Managing the Ingress ALB lifecycle](/docs/containers?topic=containers-ingress-types).
{: shortdesc}



## Updating managed add-ons
{: #addons}

Managed {{site.data.keyword.containerlong_notm}} add-ons are an easy way to enhance your cluster with open-source capabilities, such as Istio. The version of the open-source tool that you add to your cluster is tested by IBM and approved for use in {{site.data.keyword.containerlong_notm}}. To update managed add-ons that you enabled in your cluster to the latest versions, see [Updating managed add-ons](/docs/openshift?topic=openshift-managed-addons#updating-managed-add-ons).









