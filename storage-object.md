---

copyright: 
  years: 2014, 2022
lastupdated: "2022-02-09"

keywords: openshift

subcollection: openshift


---

{{site.data.keyword.attribute-definition-list}}


# Storing data on {{site.data.keyword.cos_full_notm}}
{: #object_storage}

[{{site.data.keyword.cos_full_notm}}](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage) is persistent, highly available storage that you can mount to your apps. The plug-in is a Kubernetes Flex-Volume plug-in that connects Cloud {{site.data.keyword.cos_short}} buckets to pods in your cluster. Information stored with {{site.data.keyword.cos_full_notm}} is encrypted in transit and at rest, dispersed across many geographic locations, and accessed over HTTP by using a REST API.
{: shortdesc}

If you want to use {{site.data.keyword.cos_full_notm}} in a private cluster without public network access, you must set up your {{site.data.keyword.cos_full_notm}} service instance for HMAC authentication. If you don't want to use HMAC authentication, you must open up all outbound network traffic on port 443 for the plug-in to work properly in a private cluster.
{: important}

## Creating your object storage service instance
{: #create_cos_service}

Before you can start using object storage in your cluster, you must provision an {{site.data.keyword.cos_full_notm}} service instance in your account.
{: shortdesc}

The {{site.data.keyword.cos_full_notm}} plug-in works with any s3 API endpoint. For example, you might want to use a local Cloud Object Storage server, such as [Minio](https://min.io/){: external}, or connect to an s3 API endpoint that you set up at a different cloud provider instead of an {{site.data.keyword.cos_full_notm}} service instance.

Follow these steps to create an {{site.data.keyword.cos_full_notm}} service instance. If you plan to use a local Cloud Object Storage server or a different s3 API endpoint, see the provider documentation to set up your Cloud Object Storage instance.

1. Open the [{{site.data.keyword.cos_full_notm}} catalog page](https://cloud.ibm.com/objectstorage/create).
2. Enter a name for your service instance, such as `cos-backup`, and select the same resource group that your cluster is in. To view the resource group of your cluster, run `ibmcloud oc cluster get --cluster <cluster_name_or_ID>`.   
3. Review the [plan options](https://cloud.ibm.com/objectstorage/create#pricing){: external} for pricing information and select a plan.
4. Click **Create**. The service details page opens.
5. To continue to set up {{site.data.keyword.cos_short}} to use with your cluster, see [Creating service credentials](#service_credentials).

## Creating {{site.data.keyword.cos_full_notm}} service credentials
{: #service_credentials}

Before you begin, [Create your object storage service instance](#create_cos_service).

1. In the navigation on the service details page for your {{site.data.keyword.cos_short}} instance, click **Service Credentials**.
2. Click **New credential**. A dialog box opens.
3. Enter a name for your credentials.
4. From the **Role** drop-down list, select the [{{site.data.keyword.cos_short}} access role](/docs/cloud-object-storage?topic=cloud-object-storage-iam#iam-roles) for the actions that you want storage users in your cluster to have access to. You must select at least **Writer** service access role to use the `auto-create-bucket` dynamic provisioning feature. If you select `Reader`, then you can't use the credentials to create buckets in {{site.data.keyword.cos_full_notm}} and write data to it.
5. Optional: In **Add Inline Configuration Parameters (Optional)**, enter `{"HMAC":true}` to create more HMAC credentials for the {{site.data.keyword.cos_full_notm}} service. HMAC authentication adds an extra layer of security to the OAuth2 authentication by preventing the misuse of expired or randomly created OAuth2 tokens.
6. Click **Add**. Your new credentials are in the **Service Credentials** table.
7. Click **View credentials**.
8. Make note of the **apikey** to use OAuth2 tokens to authenticate with the {{site.data.keyword.cos_full_notm}} service. For HMAC authentication, in the **cos_hmac_keys** section, note the **access_key_id** and the **secret_access_key**.
9. [Store your service credentials in a Kubernetes secret inside the cluster](#create_cos_secret) to enable access to your {{site.data.keyword.cos_full_notm}} service instance.



## Creating a secret for the object storage service credentials
{: #create_cos_secret}

To access your {{site.data.keyword.cos_full_notm}} service instance to read and write data, you must securely store the service credentials in a Kubernetes secret. The {{site.data.keyword.cos_full_notm}} plug-in uses these credentials for every read or write operation to your bucket.
{: shortdesc}

Follow these steps to create a Kubernetes secret for the credentials of an {{site.data.keyword.cos_full_notm}} service instance. If you plan to use a local Cloud Object Storage server or a different s3 API endpoint, create a Kubernetes secret with the appropriate credentials.

Before you begin:
* [Access your {{site.data.keyword.openshiftshort}} cluster](/docs/openshift?topic=openshift-access_cluster).
* Make sure that you have the **Manager** service access role for the cluster.

To create a secret for your {{site.data.keyword.cos_full_notm}} credentials:

1. Retrieve the **apikey**, or the **access_key_id** and the **secret_access_key** of your [{{site.data.keyword.cos_full_notm}} service credentials](#service_credentials). Note that the service credentials must be enough for the bucket operations that your app needs to perform. For example, if your app reads data from a bucket, the service credentials you see in your secret must have **Reader** permissions at minimum.

2. Get the **GUID** of your {{site.data.keyword.cos_full_notm}} service instance.
    ```sh
    ibmcloud resource service-instance <service_name> | grep GUID
    ```
    {: pre}

3. Create a Kubernetes secret to store your service credentials. When you create your secret, all values are automatically encoded to base64. In the following example the secret name is `cos-write-access`.

    Example `create secret` command that uses an API key.
    ```sh
    oc create secret generic cos-write-access --type=ibm/ibmc-s3fs --from-literal=api-key=<api_key> --from-literal=service-instance-id=<service_instance_guid>
    ```
    {: pre}

    Example `create secret` command that uses HMAC authentication.
    ```sh
    oc create secret generic cos-write-access --type=ibm/ibmc-s3fs --from-literal=access-key=<access_key_ID> --from-literal=secret-key=<secret_access_key>    
    ```
    {: pre}

    `api-key`
    :   Enter the API key that you retrieved from your {{site.data.keyword.cos_full_notm}} service credentials earlier. If you want to use HMAC authentication, specify the `access-key` and `secret-key` instead.
    
    `access-key`
    :   Enter the access key ID that you retrieved from your {{site.data.keyword.cos_full_notm}} service credentials earlier. If you want to use OAuth2 authentication, specify the `api-key` instead.
    
    `secret-key`
    :   Enter the secret access key that you retrieved from your {{site.data.keyword.cos_full_notm}} service credentials earlier. If you want to use OAuth2 authentication, specify the `api-key` instead.
    
    `service-instance-id`
    :   Enter the GUID of your {{site.data.keyword.cos_full_notm}} service instance that you retrieved earlier.

4. Get the secrets in your cluster and verify the output.
    ```sh
    oc get secret
    ```
    {: pre}

    Example output

    ```sh
    NAME                  TYPE                                  DATA   AGE
    cos-write-access      ibm/ibmc-s3fs                         2      7d19h
    default-au-icr-io     kubernetes.io/dockerconfigjson        1      55d
    default-de-icr-io     kubernetes.io/dockerconfigjson        1      55d
    ...
    ```
    {: screen}

5. [Install the {{site.data.keyword.cos_full_notm}} plug-in](#install_cos), or if you already installed the plug-in, [decide on the configuration]( #configure_cos) for your {{site.data.keyword.cos_full_notm}} bucket.

6. **Optional**: [Add your secret to the default storage classes](#storage_class_custom).




## Installing the {{site.data.keyword.cos_full_notm}} plug-in
{: #install_cos}

Install the {{site.data.keyword.cos_full_notm}} plug-in with a Helm chart to set up pre-defined storage classes for {{site.data.keyword.cos_full_notm}}. You can use these storage classes to create a PVC to provision {{site.data.keyword.cos_full_notm}} for your apps.
{: shortdesc}

Looking for instructions for how to update or remove the {{site.data.keyword.cos_full_notm}} plug-in? See [Updating the plug-in](#update_cos_plugin) and [Removing the plug-in](#remove_cos_plugin).
{: tip}

The {{site.data.keyword.cos_full_notm}} plug-in requires at least 0.2 vCPU and 128 MB of memory.
{: note}

Before you begin: [Access your {{site.data.keyword.openshiftshort}} cluster](/docs/openshift?topic=openshift-access_cluster).

To install the `ibmc` Helm plug-in and the `ibm-object-storage-plugin`:

1. Make sure that your worker node applies the latest patch for your minor version to run your worker node with the latest security settings. The patch version also ensures that the root password on the worker node is renewed. 

    If you did not apply updates or reload your worker node within the last 90 days, your root password on the worker node expires and the installation of the storage plug-in might fail. 
    {: note}

    1. List the current patch version of your worker nodes.
        ```sh
        ibmcloud oc worker ls --cluster <cluster_name_or_ID>
        ```
        {: pre}

        Example output
        ```sh
        OK
        ID                                                  Public IP        Private IP     Machine Type           State    Status   Zone    Version
        kube-dal10-crb1a23b456789ac1b20b2nc1e12b345ab-w26   169.xx.xxx.xxx    10.xxx.xx.xxx   b3c.4x16.encrypted     normal   Ready    dal10   1.21.6_1523*
        ```
        {: screen}

        If your worker node does not apply the latest patch version, you see an asterisk (`*`) in the **Version** column of your CLI output.

    2. Review the [version changelog](/docs/containers?topic=containers-changelog) to find the changes that are in the latest patch version.

    3. Apply the latest patch version by reloading your worker node. Follow the instructions in the [ibmcloud oc worker reload command](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_worker_reload) to gracefully reschedule any running pods on your worker node before you reload your worker node. Note that during the reload, your worker node machine is updated with the latest image and data is deleted if not [stored outside the worker node](/docs/openshift?topic=openshift-storage_planning#persistent_storage_overview).
1. Review the changelog and verify support for your [cluster version and architecture](/docs/containers?topic=containers-cos_plugin_changelog).
2. [Follow the instructions](/docs/openshift?topic=openshift-helm#install_v3) to install the version 3 Helm client on your local machine.

    If you enabled [VRF](/docs/account?topic=account-vrf-service-endpoint#vrf) and [service endpoints](/docs/account?topic=account-vrf-service-endpoint#service-endpoint) in your {{site.data.keyword.cloud_notm}} account, you can use the private {{site.data.keyword.cloud_notm}} Helm repository to keep your image pull traffic on the private network. If you can't enable VRF or service endpoints in your account, use the public Helm repository.
    {: note}

3. Add the {{site.data.keyword.cloud_notm}} Helm repo to your cluster.

    ```sh
    helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm
    ```
    {: pre}

4. Update the Helm repo to retrieve the most recent version of all Helm charts in this repo.
    ```sh
    helm repo update
    ```
    {: pre}

5. If you installed the {{site.data.keyword.cos_full_notm}} Helm plug-in earlier, remove the `ibmc` plug-in.
    ```sh
    helm plugin uninstall ibmc
    ```
    {: pre}

6. Download the Helm charts and unpack the charts in your current directory.

    ```sh
    helm fetch --untar ibm-helm/ibm-object-storage-plugin
    ```
    {: pre}

    If the output shows `Error: failed to untar: a file or directory with the name ibm-object-storage-plugin already exists`, delete your `ibm-object-storage-plugin` directory and rerun the `helm fetch` command.
    {: tip}

7. If you use OS X or a Linux distribution, install the {{site.data.keyword.cos_full_notm}} Helm plug-in `ibmc`. The plug-in automatically retrieves your cluster location and to set the API endpoint for your {{site.data.keyword.cos_full_notm}} buckets in your storage classes. If you use Windows as your operating system, continue with the next step.
    1. Install the Helm plug-in.
        ```sh
        helm plugin install ./ibm-object-storage-plugin/helm-ibmc
        ```
        {: pre}

    2. Verify that the `ibmc` plug-in is installed successfully.
        ```sh
        helm ibmc --help
        ```
        {: pre}

        If the output shows the error `Error: fork/exec /home/iksadmin/.helm/plugins/helm-ibmc/ibmc.sh: permission denied`, run `chmod 755 /Users/<user_name>/Library/helm/plugins/helm-ibmc/ibmc.sh`. Then, rerun `helm ibmc --help`.
        {: tip}

        Example output

        ```sh
        helm version: v3.2.4+g0ad800e
        Install or upgrade Helm charts in IBM K8S Service(IKS) and IBM Cloud Private(ICP)
        Usage:
            helm ibmc [command]
        Available Commands:
            install           Install a Helm chart
            upgrade           Upgrade the release to a new version of the Helm chart
        Available Flags:
            -h, --help        (Optional) This text.
            -u, --update      (Optional) Update this plugin to the latest version
        Example Usage:
            Install: helm ibmc install ibm-object-storage-plugin ibm-helm/ibm-object-storage-plugin
            Upgrade: helm ibmc upgrade [RELEASE] ibm-helm/ibm-object-storage-plugin
        Note:
            1. It is always recommended to install latest version of ibm-object-storage-plugin chart.
            2. It is always recommended to have 'kubectl' client up-to-date.
        ```
        {: screen}

8. Optional: Limit the {{site.data.keyword.cos_full_notm}} plug-in to access only the Kubernetes secrets that hold your {{site.data.keyword.cos_full_notm}} service credentials. By default, the plug-in can access all Kubernetes secrets in your cluster.
    1. [Create your {{site.data.keyword.cos_full_notm}} service instance](#create_cos_service).
    2. [Store your {{site.data.keyword.cos_full_notm}} service credentials in a Kubernetes secret](#create_cos_secret).
    3. From the `ibm-object-storage-plugin`, navigate to the `templates` directory and list available files.
        **OS X and Linux**

        ```sh
        cd templates && ls
        ```
        {: pre}

        **Windows**

        ```sh
        chdir templates && dir
        ```
        {: pre}

    4. Open the `provisioner-sa.yaml` file and look for the `ibmcloud-object-storage-secret-reader` `ClusterRole` definition.
    5. Add the name of the secret that you created earlier to the list of secrets that the plug-in is authorized to access in the `resourceNames` section.
        ```yaml
        kind: ClusterRole
        apiVersion: rbac.authorization.k8s.io/v1beta1
        metadata:
         name: ibmcloud-object-storage-secret-reader
        rules:
        - apiGroups: [""]
          resources: ["secrets"]
          resourceNames: ["<secret_name1>","<secret_name2>"]
          verbs: ["get"]
        ```
        {: codeblock}

    6. Save your changes and navigate to your working directory.

9. Install the `ibm-object-storage-plugin` in your cluster. When you install the plug-in, pre-defined storage classes are added to your cluster. If you completed the previous step for limiting the {{site.data.keyword.cos_full_notm}} plug-in to access only the Kubernetes secrets that hold your {{site.data.keyword.cos_full_notm}} service credentials and you are still targeting the `templates` directory, change directories to your working directory. **VPC clusters only**: To enable authorized IPs on VPC, set the `--set bucketAccessPolicy=true` flag.

Example `helm ibmc install` command for OS X and Linux.

```sh
helm ibmc install ibm-object-storage-plugin ibm-helm/ibm-object-storage-plugin --set license=true [--set bucketAccessPolicy=false]
```
{: pre}

Example `helm install` command for Windows.

```sh
helm install ibm-object-storage-plugin ./ibm-object-storage-plugin --set dcname="${DC_NAME}" --set provider="${CLUSTER_PROVIDER}" --set workerOS="${WORKER_OS}" --region="${REGION} --set platform="${PLATFORM}" --set license=true [--set bucketAccessPolicy=false]
```
{: pre}



`DC_NAME`
:   The cluster data center. To retrieve the data center, run `oc get cm cluster-info -n kube-system -o jsonpath="{.data.cluster-config\.json}{'\n'}"`. Store the data center value in an environment variable by running `SET DC_NAME=<datacenter>`. Optional: Set the environment variable in Windows PowerShell by running `$env:DC_NAME="<datacenter>"`.

`CLUSTER_PROVIDER`
:   The infrastructure provider. To retrieve this value, run `oc get nodes -o jsonpath="{.items[*].metadata.labels.ibm-cloud\.kubernetes\.io\/iaas-provider}{'\n'}"`. If the output from the previous step contains `softlayer`, then set the `CLUSTER_PROVIDER` to `"IBMC"`. If the output contains `gc`, `ng`, or `g2`, then set the `CLUSTER_PROVIDER` to `"IBMC-VPC"`. Store the infrastructure provider in an environment variable. For example: `SET CLUSTER_PROVIDER="IBMC-VPC"`.

`WORKER_OS` and `PLATFORM`
:   The operating system of the worker nodes. To retrieve these values, run `oc get nodes -o jsonpath="{.items[*].metadata.labels.ibm-cloud\.kubernetes\.io\/os}{'\n'}"`. Store the operating system of the worker nodes in an environment variable. For {{site.data.keyword.openshiftlong_notm}} clusters, run `SET WORKER_OS="redhat"` and `SET PLATFORM="openshift"`. 

`REGION`
:   The region of the worker nodes. To retrieve this value, run `oc get nodes -o yaml | grep 'ibm-cloud\.kubernetes\.io/region'`. Store the region of the worker nodes in an environment variable by running `SET REGION="< region>"`. |


### Verifying your installation
{: #cos-plugin verify}

Review the pod details to verify that the plug-in installation succeeded.
{: shortdesc}

1. Verify the installation succeeded by listing the driver pods.

    ```sh
    oc get pod --all-namespaces -o wide | grep object
    ```
    {: pre}

    Example output

    ```sh
    ibmcloud-object-storage-driver-9n8g8                              1/1       Running   0          2m
    ibmcloud-object-storage-plugin-7c774d484b-pcnnx                   1/1       Running   0          2m
    ```
    {: screen}

    The installation is successful when you see one `ibmcloud-object-storage-plugin` pod and one or more `ibmcloud-object-storage-driver` pods. The number of `ibmcloud-object-storage-driver` pods equals the number of worker nodes in your cluster. All pods must be in a `Running` state for the plug-in to function properly. If the pods fail, run `oc describe pod -n ibm-object-s3fs <pod_name>` to find the root cause for the failure.

1. Verify that the storage classes are created successfully.

    ```sh
    oc get storageclass | grep s3
    ```
    {: pre}

    Example output

    ```sh
    ibmc-s3fs-cold-cross-region            ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-cold-regional                ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-cross-region            ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-perf-cross-region       ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-perf-regional           ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-regional                ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-cross-region        ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-perf-cross-region   ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-perf-regional       ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-regional            ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-vault-cross-region           ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-vault-regional               ibm.io/ibmc-s3fs   8m
    ```
    {: screen}

    If you want to set one of the {{site.data.keyword.cos_full_notm}} storage classes as your default storage class, run `oc patch storage class <storageclass> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'`. Replace `<storageclass>` with the name of the {{site.data.keyword.cos_full_notm}} storage class.
    {: tip}

1. Follow the instructions to [add object storage to your apps](#add_cos).

If you're having trouble installing the {{site.data.keyword.cos_full_notm}} plug-in, see [Object storage: Installing the Object storage `ibmc` Helm plug-in fails](/docs/containers?topic=containers-cos_helm_fails) and [Object storage: Installing the {{site.data.keyword.cos_full_notm}} plug-in fails](/docs/containers?topic=containers-cos_plugin_fails).
{: tip}

### Updating the {{site.data.keyword.cos_full_notm}} plug-in
{: #update_cos_plugin}

You can upgrade the existing {{site.data.keyword.cos_full_notm}} plug-in to the most recent version.
{: shortdesc}


1. Get the name of your {{site.data.keyword.cos_full_notm}} plug-in Helm release and the version of the plug-in in your cluster.

    ```sh
    helm ls -A | grep object
    ```
    {: pre}

    Example output

    ```sh
    NAME              NAMESPACE      REVISION    UPDATED                                   STATUS      CHART                                  APP VERSION               
    <release_name>  <namespace>     1           2020-02-13 16:05:58.599679 -0500 EST    deployed    ibm-object-storage-plugin-1.1.2        1.1.2
    ```
    {: screen}

1. Follow the steps in [Installing the {{site.data.keyword.cos_full_notm}} plug-in](#install_cos) to install the latest version of the {{site.data.keyword.cos_full_notm}} plug-in.

1. Update the {{site.data.keyword.cloud_notm}} Helm repo to retrieve the most recent version of all Helm charts in this repo.

    ```sh
    helm repo update
    ```
    {: pre}

1. Update the {{site.data.keyword.cos_full_notm}} `ibmc` Helm plug-in to the most recent version.

    ```sh
    helm ibmc --update
    ```
    {: pre}

1. Install the most recent version of the `ibm-object-storage-plugin` for your operating system.

  Example `helm ibmc install` command for OS X and Linux.

    ```sh
    helm ibmc install ibm-object-storage-plugin ibm-helm/ibm-object-storage-plugin --set license=true [--set bucketAccessPolicy=false]
    ```
    {: pre}

  Example `helm install` command for Windows.

    ```sh
    helm install ibm-object-storage-plugin ./ibm-object-storage-plugin --set dcname="${DC_NAME}" --set provider="${CLUSTER_PROVIDER}" --set workerOS="${WORKER_OS}" --region="${REGION} --set platform="${PLATFORM}" --set license=true [--set bucketAccessPolicy=false]
    ```
    {: pre}

    `DC_NAME`
    :   The cluster data center. To retrieve the data center, run `oc get cm cluster-info -n kube-system -o jsonpath="{.data.cluster-config\.json}{'\n'}"`. Store the data center value in an environment variable by running `SET DC_NAME=<datacenter>`. Optional: Set the environment variable in Windows PowerShell by running `$env:DC_NAME="<datacenter>"`.

    `CLUSTER_PROVIDER`
    :   The infrastructure provider. To retrieve this value, run `oc get nodes -o jsonpath="{.items[*].metadata.labels.ibm-cloud\.kubernetes\.io\/iaas-provider}{'\n'}"`. If the output from the previous step contains `softlayer`, then set the `CLUSTER_PROVIDER` to `"IBMC"`. If the output contains `gc`, `ng`, or `g2`, then set the `CLUSTER_PROVIDER` to `"IBMC-VPC"`. Store the infrastructure provider in an environment variable. For example: `SET CLUSTER_PROVIDER="IBMC-VPC"`.

    `WORKER_OS` and `PLATFORM`
    :   The operating system of the worker nodes. To retrieve these values, run `oc get nodes -o jsonpath="{.items[*].metadata.labels.ibm-cloud\.kubernetes\.io\/os}{'\n'}"`. Store the operating system of the worker nodes in an environment variable. For {{site.data.keyword.openshiftlong_notm}} clusters, run `SET WORKER_OS="redhat"` and `SET PLATFORM="openshift"`. 

    `REGION`
    :   The region of the worker nodes. To retrieve this value, run `oc get nodes -o yaml | grep 'ibm-cloud\.kubernetes\.io/region'`. Store the region of the worker nodes in an environment variable by running `SET REGION="< region>"`. |

1. Verify that the `ibmcloud-object-storage-plugin` is successfully upgraded. The upgrade of the plug-in is successful when you see `deployment "ibmcloud-object-storage-plugin" successfully rolled out` in your CLI output.

    ```sh
    oc rollout status deployment/ibmcloud-object-storage-plugin -n ibm-object-s3fs
    ```
    {: pre}

1. Verify that the `ibmcloud-object-storage-driver` is successfully upgraded. The upgrade is successful when you see `daemon set "ibmcloud-object-storage-driver" successfully rolled out` in your CLI output.

    ```sh
    oc rollout status ds/ibmcloud-object-storage-driver -n ibm-object-s3fs
    ```
    {: pre}

1. Verify that the {{site.data.keyword.cos_full_notm}} pods are in a `Running` state.

    ```sh
    oc get pods -n <namespace> -o wide | grep object-storage
    ```
    {: pre}

If you're having trouble updating the {{site.data.keyword.cos_full_notm}} plug-in, see [Object storage: Installing the Object storage `ibmc` Helm plug-in fails](/docs/containers?topic=containers-cos_helm_fails) and [Object storage: Installing the {{site.data.keyword.cos_full_notm}} plug-in fails](/docs/containers?topic=containers-cos_plugin_fails).
{: tip}

### Removing the {{site.data.keyword.cos_full_notm}} plug-in
{: #remove_cos_plugin}

If you don't want to provision and use {{site.data.keyword.cos_full_notm}} in your cluster, you can uninstall the `ibm-object-storage-plugin` and the `ibmc` Helm plug-in.
{: shortdesc}

Removing the `ibmc` Helm plug-in or the `ibm-object-storage-plugin` doesn't remove existing PVCs, PVs, data. When you remove the `ibm-object-storage-plugin`, all the related pods and daemon sets are removed from your cluster. You can't provision new {{site.data.keyword.cos_full_notm}} for your cluster or use existing PVCs and PVs after you remove the plug-in, unless you configure your app to use the {{site.data.keyword.cos_full_notm}} API directly.
{: important}

Before you begin:

- [Access your {{site.data.keyword.openshiftshort}} cluster](/docs/openshift?topic=openshift-access_cluster).
- Make sure that you don't have any PVCs or PVs in your cluster that use {{site.data.keyword.cos_full_notm}}. To list all pods that mount a specific PVC, run `oc get pods --all-namespaces -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.volumes[*]}{.persistentVolumeClaim.claimName}{" "}{end}{end}' | grep "<pvc_name>"`.

To remove the `ibmc` Helm plug-in and the `ibm-object-storage-plugin`:

1. Get the name of your `ibm-object-storage-plugin` Helm installation.

    ```sh
    helm ls -A | grep ibm-object-storage-plugin
    ```
    {: pre}

    Example output

    ```sh
    NAME                         NAMESPACE    REVISION    UPDATED                                 STATUS      CHART                                  APP VERSION               
    ibm-object-storage-plugin    default      2           2020-04-01 08:46:01.403477 -0400 EDT    deployed    ibm-object-storage-plugin-1.1.4        1.1.4  
    ```
    {: screen}

2. Uninstall the `ibm-object-storage-plugin`.

    ```sh
    helm uninstall <release_name>
    ```
    {: pre}

    Example command for a release named `ibm-object-storage-plugin`.

    ```sh
    helm uninstall ibm-object-storage-plugin
    ```
    {: pre}

3. Verify that the `ibm-object-storage-plugin` pods are removed.

    ```sh
    oc get pod -n <namespace> | grep object-storage
    ```
    {: pre}

    The removal of the pods is successful if no pods are displayed in your CLI output. The removal of the storage classes is successful if no storage classes are displayed in your CLI output.

4. Verify that the storage classes are removed.

    ```sh
    oc get storageclasses | grep s3
    ```
    {: pre}

5. If you use OS X or a Linux distribution, remove the `ibmc` Helm plug-in. If you use Windows, this step is not required.

    1. Remove the `ibmc` Helm plug-in.

        ```sh
        helm plugin uninstall ibmc
        ```
        {: pre}

    2. Verify that the `ibmc` plug-in is removed. The `ibmc` plug-in is removed successfully if the `ibmc` plug-in is not listed in your CLI output.

        ```sh
        helm plugin list
        ```
        {: pre}

        Example output

        ```sh
        NAME    VERSION    DESCRIPTION
        ```
        {: screen}

## Deciding on the object storage configuration
{: #configure_cos}

{{site.data.keyword.openshiftlong_notm}} provides pre-defined storage classes that you can use to create buckets with a specific configuration.
{: shortdesc}

1. List available storage classes in {{site.data.keyword.openshiftlong_notm}}.

    ```sh
    oc get storageclasses | grep s3
    ```
    {: pre}

    Example output

    ```sh
    ibmc-s3fs-cold-cross-region            ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-cold-regional                ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-cross-region            ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-perf-cross-region       ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-perf-regional           ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-flex-regional                ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-cross-region        ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-perf-cross-region   ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-perf-regional       ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-standard-regional            ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-vault-cross-region           ibm.io/ibmc-s3fs   8m
    ibmc-s3fs-vault-regional               ibm.io/ibmc-s3fs   8m
    ```
    {: screen}

2. Choose a storage class that fits your data access requirements. The storage class determines the [pricing](https://cloud.ibm.com/objectstorage/create#pricing){: external} for storage capacity, read and write operations, and outbound bandwidth for a bucket. The option that is right for you is based on how frequently data is read and written to your service instance.
    - **Standard**: This option is used for hot data that is accessed frequently. Common use cases are web or mobile apps.
    - **Vault**: This option is used for workloads or cool data that are accessed infrequently, such as once a month or less. Common use cases are archives, short-term data retention, digital asset preservation, tape replacement, and disaster recovery.
    - **Cold**: This option is used for cold data that is rarely accessed (every 90 days or less), or inactive data. Common use cases are archives, long-term backups, historical data that you keep for compliance, or workloads and apps that are rarely accessed.
    - **Flex**: This option is used for workloads and data that don't follow a specific usage pattern, or that are too huge to determine or predict a usage pattern. **Tip:** Check out this [blog](https://www.ibm.com/blogs/cloud-archive/2017/03/interconnect-2017-changing-rules-storage/){: external} to learn how the Flex storage class works compared to traditional storage tiers.   

3. Decide on the level of resiliency for the data that is stored in your bucket.
    - **Cross-region**: With this option, your data is stored across three regions within a geolocation for highest availability. If you have workloads that are distributed across regions, requests are routed to the nearest regional endpoint. The API endpoint for the geolocation is automatically set by the `ibmc` Helm plug-in that you installed earlier based on the location that your cluster is in. For example, if your cluster is in `US South`, then your storage classes are configured to use the `US GEO` API endpoint for your buckets. For more information, see [Regions and endpoints](/docs/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#endpoints).  
    - **Regional**: With this option, your data is replicated across multiple zones within one region. If you have workloads that are located in the same region, you see lower latency and better performance than in a cross-regional setup. The regional endpoint is automatically set by the `ibm` Helm plug-in that you installed earlier based on the location that your cluster is in. For example, if your cluster is in `US South`, then your storage classes were configured to use `US South` as the regional endpoint for your buckets. For more information, see [Regions and endpoints](/docs/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#endpoints).

4. Review the detailed {{site.data.keyword.cos_full_notm}} bucket configuration for a storage class.

    ```sh
    oc describe storageclass <storageclass_name>
    ```
    {: pre}

    Example output

    ```sh
    Name:                  ibmc-s3fs-standard-cross-region
    IsDefaultClass:        No
    Annotations:           <none>
    Provisioner:           ibm.io/ibmc-s3fs
    Parameters:            ibm.io/chunk-size-mb=16,ibm.io/curl-debug=false,ibm.io/debug-level=warn,ibm.io/iam-endpoint=https://iam.bluemix.net,ibm.io/kernel-cache=true,ibm.io/multireq-max=20,ibm.io/object-store-endpoint=https://s3-api.dal-us-geo.objectstorage.service.networklayer.com,ibm.io/object-store-storage-class=us-standard,ibm.io/parallel-count=2,ibm.io/s3fs-fuse-retry-count=5,ibm.io/stat-cache-size=100000,ibm.io/tls-cipher-suite=AESGCM
    AllowVolumeExpansion:  <unset>
    MountOptions:          <none>
    ReclaimPolicy:         Delete
    VolumeBindingMode:     Immediate
    Events:                <none>
    ```
    {: screen}


    `ibm.io/chunk-size-mb`
    :   The size of a data chunk that is read from or written to {{site.data.keyword.cos_full_notm}} in megabytes. Storage classes with `perf` in their name are set up with 52 megabytes. Storage classes without `perf` in their name use 16 megabyte chunks. For example, if you want to read a file that is `1GB`, the plug-in reads this file in multiple 16 or 52-megabyte chunks.
    
    `ibm.io/curl-debug`
    :   Enable the logging of requests that are sent to the {{site.data.keyword.cos_full_notm}} service instance. If enabled, logs are sent to `syslog` and you can [forward the logs to an external logging server](/docs/containers?topic=containers-health#logging). By default, all storage classes are set to `false` to disable this logging feature.
    
    `ibm.io/debug-level`
    :   The logging level that is set by the {{site.data.keyword.cos_full_notm}} plug-in. All storage classes are set up with the `WARN` logging level.
    
    `ibm.io/iam-endpoint`
    :   The API endpoint for {{site.data.keyword.cloud_notm}} Identity and Access Management (IAM). 
    
    `ibm.io/kernel-cache`
    :   Enable or disable the kernel buffer cache for the volume mount point. If enabled, data that is read from {{site.data.keyword.cos_full_notm}} is stored in the kernel cache to ensure fast read access to your data. If disabled, data is not cached and always read from {{site.data.keyword.cos_full_notm}}. Kernel cache is enabled for `standard` and `flex` storage classes, and disabled for `cold` and `vault` storage classes. 
    
    `ibm.io/multireq-max`
    :   The maximum number of parallel requests that can be sent to the {{site.data.keyword.cos_full_notm}} service instance to list files in a single directory. All storage classes are set up with a maximum of 20 parallel requests.
    
    `ibm.io/object-store-endpoint`
    :   The API endpoint to use to access the bucket in your {{site.data.keyword.cos_full_notm}} service instance. The endpoint is automatically set based on the region of your cluster. If you want to access an existing bucket that is located in a different region than the one where your cluster is in, you must [create a custom storage class](/docs/openshift?topic=openshift-kube_concepts#customized_storageclass) and use the API endpoint for your bucket.
    
    `ibm.io/object-store-storage-class`
    :   The name of the storage class.
    
    `ibm.io/parallel-count`
    :   The maximum number of parallel requests that can be sent to the {{site.data.keyword.cos_full_notm}} service instance for a single read or write operation. Storage classes with `perf` in their name are set up with a maximum of 20 parallel requests. Storage classes without `perf` are set up with two parallel requests by default.
    
    `ibm.io/s3fs-fuse-retry-count`
    :   The maximum number of retries for a read or write operation before the operation is considered unsuccessful. All storage classes are set up with a maximum of five retries.
    
    `ibm.io/stat-cache-size`
    :   The maximum number of records that are kept in the {{site.data.keyword.cos_full_notm}} metadata cache. Every record can take up to 0.5 kilobytes. All storage classes set the maximum number of records to 100000 by default.
    
    `ibm.io/tls-cipher-suite`
    :   The TLS cipher suite that must be used when a connection to {{site.data.keyword.cos_full_notm}} is established via the HTTPS endpoint. The value for the cipher suite must follow the [OpenSSL format](https://www.openssl.org/docs/man1.1.1/man1/ciphers.html){: external}. If your worker nodes run an Ubuntu operating system, your storage classes are set up to use the `AESGCM`cipher suite by default. For worker nodes that run a Red Hat operating system, the `ecdhe_rsa_aes_128_gcm_sha_256` cipher suite is used by default.


    For more information about each storage class, see the [storage class reference](#cos_storageclass_reference). If you want to change any of the pre-set values, create your own [customized storage class](/docs/openshift?topic=openshift-kube_concepts#customized_storageclass).
    {: tip}

5. Decide on a name for your bucket. The name of a bucket must be unique in {{site.data.keyword.cos_full_notm}}. You can also choose to automatically create a name for your bucket by the {{site.data.keyword.cos_full_notm}} plug-in. To organize data in a bucket, you can create subdirectories.

    The storage class that you chose earlier determines the pricing for the entire bucket. You can't define different storage classes for subdirectories. If you want to store data with different access requirements, consider creating multiple buckets by using multiple PVCs.
    {: note}

6. Choose if you want to keep your data and the bucket after the cluster or the persistent volume claim (PVC) is deleted. When you delete the PVC, the PV is always deleted. You can choose if you want to also automatically delete the data and the bucket when you delete the PVC. Your {{site.data.keyword.cos_full_notm}} service instance is independent from the retention policy that you select for your data and is never removed when you delete a PVC.

Now that you decided on the configuration that you want, you are ready to [create a PVC](#add_cos) to provision {{site.data.keyword.cos_full_notm}}.



## VPC: Setting up authorized IP addresses for {{site.data.keyword.cos_full_notm}}
{: #cos_auth_ip}

You can authorize your VPC Cloud Service Endpoint source IP addresses to access your {{site.data.keyword.cos_full_notm}} bucket. When you set up authorized IP addresses, you can only access your bucket data from those IP addresses; for example, in an app pod.
{: shortdesc}

Minimum required permissions
:   **Manager** service access role for the {{site.data.keyword.openshiftlong_notm}} service.
:   **Writer** service access role for the {{site.data.keyword.cos_full_notm}} service.

**Supported infrastructure provider**:
* ![VPC infrastructure provider icon.](images/icon-vpc-2.svg) VPC

1. [Follow the instructions to install the `ibmc` Helm plug-in](#install_cos). Make sure to install the `ibm-object-storage-plugin` and set the `bucketAccessPolicy` flag to `true`.

2. Create one `Manager` HMAC service credential and one `Writer` HMAC service credential for your {{site.data.keyword.cos_full_notm}} instance.
    * [Creating HMAC credentials from the console](/docs/cloud-object-storage?topic=cloud-object-storage-uhc-hmac-credentials-main).
    * [Creating HMAC credentials from the CLI](/docs/cloud-object-storage?topic=cloud-object-storage-uhc-hmac-credentials-main#uhc-create-hmac-credentials-cli).

3. Encode the `apikey` from your {{site.data.keyword.cos_full_notm}} Manager credentials to base64.

    ```sh
    echo -n "<cos_manager_apikey>" | base64
    ```
    {: pre}

4. Encode the `access-key` and `secret-key` from your {{site.data.keyword.cos_full_notm}} Writer credentials to base64.

    ```sh
    echo -n "<cos_writer_access-key>" | base64
    echo -n "<cos_writer_secret-key>" | base64
    ```
    {: pre}

5. Create a secret configuration file with the values that you encoded. For the `access-key` and `secret-key`, enter the base64 encoded `access-key` and `secret-key` from the Writer HMAC credentials that you created. For the `res-conf-apikey`, enter the base64 encoded `apikey` from your Manager HMAC credentials.
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
        name: <secret_name>
      type: ibm/ibmc-s3fs
      data:
    access-key: # Enter your base64 encoded COS Writer access-key
    secret-key: # Enter your base64 encoded COS Writer secret-key
    res-conf-apikey: # Enter your base64 encoded COS Manager api-key
    ```
    {: codeblock}

6. Create the secret in your cluster.

    ```sh
    oc create -f secret.yaml
    ```
    {: pre}

7. Verify that the secret is created.

    ```sh
    oc get secrets
    ```
    {: pre}

8. Create a PVC that uses the secret you created. Set the `ibm.io/auto-create-bucket: "true"` and `ibm.io/auto_cache: "true"` annotations to automatically create a bucket that caches your data.

    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
        name: <pvc_name>
    annotations:
      ibm.io/auto-create-bucket: "true"
      ibm.io/auto-delete-bucket: "false"
      ibm.io/auto_cache: "true"
      ibm.io/bucket: "<bucket_name>"
      ibm.io/secret-name: "<secret_name>"
      ibm.io/secret-namespace: "<secret-namespace>" # By default, the COS plug-in searches for your secret in the same namespace where you create the PVC. If you created your secret in a namespace other than the namespace where you want to create your PVC, enter the namespace where you created your secret.
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 8Gi
      storageClassName: ibmc-s3fs-standard-regional
      volumeMode: Filesystem
    ```
    {: codeblock}

9. Get a list of the Cloud Service Endpoint source IP addresses of your VPC.

    1. Get a list of your VPCs.

        ```sh
        ibmcloud is vpcs
        ```
        {: pre}

    2. Get the details of your VPC and make a note of your Cloud Service Endpoint source IP addresses.

        ```sh
        ibmcloud is vpc <vpc_ID>
        ```
        {: pre}

        Example output
        
        ```sh                                              
        ...                                              
        Cloud Service Endpoint source IP addresses:    Zone         Address      
                                                    us-south-1   10.249.XXX.XX      
                                                    us-south-2   10.249.XXX.XX     
                                                    us-south-3   10.249.XXX.XX
        ```
        {: screen}

10. Verify that the Cloud Service Endpoint source IP addresses of your VPC are authorized in your {{site.data.keyword.cos_full_notm}} bucket.
    1. [From your {{site.data.keyword.cos_full_notm}} resource list](https://cloud.ibm.com/resources), select your {{site.data.keyword.cos_full_notm}} instance and select the bucket that you specified in your PVC.
    2. Select **Access Policies** > **Authorized IPs** and verify that the Cloud Service Endpoint source IP addresses of your VPC are displayed.

    You can't read or write to your bucket from the console. You can only access your bucket from within an app pod on your cluster.
    {: note}

11. [Create a deployment YAML that references the PVC you created](#cos_app_volume_mount).

12. Create the app in your cluster.

    ```sh
    oc create -f app.yaml
    ```
    {: pre}

13. Verify that your app pod is `Running`.

    ```sh
    oc get pods | grep <app_name>
    ```
    {: pre}

14. Verify that your volume is mounted and that you can read and write to your COS bucket.

    1. Log in to your app pod.

        ```sh
        oc exec -it <pod_name> bash
        ```
        {: pre}

    2. Verify that your COS bucket is mounted from your app pod and that you can read and write to your COS bucket. Run the disk free `df` command to see available disks in your system. Your COS bucket displays the `s3fs` file system type and the mount path that you specified in your PVC.

        ```sh
        df
        ```
        {: pre}

        In this example, the COS bucket is mounted at `/cos-vpc`.

        ```sh
        Filesystem        1K-blocks    Used    Available Use% Mounted on
        overlay           102048096 9071556     87786140  10% /
        tmpfs                 65536       0        65536   0% /dev
        tmpfs               7565792       0      7565792   0% /sys/fs/cgroup
        shm                   65536       0        65536   0% /dev/shm
        /dev/vda2         102048096 9071556     87786140  10% /etc/hosts
        s3fs           274877906944       0 274877906944   0% /cos-vpc
        tmpfs               7565792      44      7565748   1% /run/secrets/kubernetes.io/serviceaccount
        tmpfs               7565792       0      7565792   0% /proc/acpi
        tmpfs               7565792       0      7565792   0% /proc/scsi
        tmpfs               7565792       0      7565792   0% /sys/firmware
        ```
        {: screen}

    3. Change directories to the directory where your COS bucket is mounted. In this example the bucket is mounted at `/cos-vpc`.

        ```sh
        cd cos-vpc
        ```
        {: pre}

    4. Write a `test.txt` file to your COS bucket and list files to verify that the file was written.

        ```sh
        touch test.txt && ls
        ```
        {: pre}

    5. Remove the file and log out of your app pod.

        ```sh
        rm test.txt && exit
        ```
        {: pre}

## Adding object storage to apps
{: #add_cos}

Create a persistent volume claim (PVC) to provision {{site.data.keyword.cos_full_notm}} for your cluster.
{: shortdesc}

Depending on the settings that you choose in your PVC, you can provision {{site.data.keyword.cos_full_notm}} in the following ways:
- [Dynamic provisioning](/docs/openshift?topic=openshift-kube_concepts#dynamic_provisioning): When you create the PVC, the matching persistent volume (PV) and the bucket in your {{site.data.keyword.cos_full_notm}} service instance are automatically created.
- [Static provisioning](/docs/openshift?topic=openshift-kube_concepts#static_provisioning): You can reference an existing bucket in your {{site.data.keyword.cos_full_notm}} service instance in your PVC. When you create the PVC, only the matching PV is automatically created and linked to your existing bucket in {{site.data.keyword.cos_full_notm}}.

Before you begin:
- [Create and prepare your {{site.data.keyword.cos_full_notm}} service instance](#create_cos_service).
- [Create a secret to store your {{site.data.keyword.cos_full_notm}} service credentials](#create_cos_secret).
- [Decide on the configuration for your {{site.data.keyword.cos_full_notm}}](#configure_cos).

To add {{site.data.keyword.cos_full_notm}} to your cluster:

1. Create a configuration file to define your persistent volume claim (PVC). If you add your [{{site.data.keyword.cos_full_notm}} credentials to the default storage classes](#storage_class_custom), you don't need to list your secret in the PVC.

    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: <name> # Enter the name of the PVC.
      namespace: <namespace> # Enter the namespace where you want to create the PVC. The PVC must be created in the same namespace where you created the Kubernetes secret for your service credentials and where you want to run your pod.
      annotations:
        ibm.io/auto-create-bucket: "<true_or_false>"
        ibm.io/auto-delete-bucket: "<true_or_false>"
        ibm.io/bucket: "<bucket_name>"
        ibm.io/object-path: "<bucket_subdirectory>"
        ibm.io/secret-name: "<secret_name>"
        ibm.io/endpoint: "https://<s3fs_service_endpoint>"
        ibm.io/tls-cipher-suite: "default"
        ibm.io/secret-namespace: "<secret-namespace>" # By default, the COS plug-in searches for your secret in the same namespace where you create the PVC. If you created your secret in a namespace other than the namespace where you want to create your PVC, enter the namespace where you created your secret.
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: <size>
      storageClassName: <storage_class>
    ```
    {: codeblock}

    `ibm.io/auto-create-bucket`
    :   Choose between the following options. 
    :   `true`: When you create the PVC, the PV and the bucket in your {{site.data.keyword.cos_full_notm}} service instance are automatically created. Choose this option to create a new bucket in your {{site.data.keyword.cos_full_notm}} service instance. Note that the service credentials must have **Writer** permissions to automatically create the bucket.
    :   `false`: Choose this option if you want to access data in an existing bucket. When you create the PVC, the PV is automatically created and linked to the bucket that you specify in `ibm.io/bucket`.

    `ibm.io/auto-delete-bucket`
    :   Choose between the following options.
    :   `true`: Your data, the bucket, and the PV is automatically removed when you delete the PVC. Your {{site.data.keyword.cos_full_notm}} service instance remains and is not deleted. If you choose to set this option to `true`, then you must set `ibm.io/auto-create-bucket: true` and `ibm.io/bucket: ""` so that your bucket is automatically created with a name with the format `tmp-s3fs-xxxx`.
    :   `false`: When you delete the PVC, the PV is deleted automatically, but your data and the bucket in your {{site.data.keyword.cos_full_notm}} service instance remain. To access your data, you must create a new PVC with the name of your existing bucket.
    
    `ibm.io/bucket`
    :   Choose between the following options.
    :   If `ibm.io/auto-create-bucket` is set to `true`: Enter the name of the bucket that you want to create in {{site.data.keyword.cos_full_notm}}. If in addition `ibm.io/auto-delete-bucket` is set to `true`, you must leave this field blank to automatically assign your bucket a name with the format `tmp-s3fs-xxxx`. The name must be unique in {{site.data.keyword.cos_full_notm}}.
    :   If `ibm.io/auto-create-bucket` is set to `false`: Enter the name of the existing bucket that you want to access in the cluster. 
    
    `ibm.io/object-path`
    :   Optional: Enter the name of the existing subdirectory in your bucket that you want to mount. Use this option if you want to mount a subdirectory only and not the entire bucket. To mount a subdirectory, you must set `ibm.io/auto-create-bucket: "false"` and provide the name of the bucket in `ibm.io/bucket`.
    
    `ibm.io/secret-name`
    :   Enter the name of the secret that holds the {{site.data.keyword.cos_full_notm}} credentials that you created earlier. If you add your [{{site.data.keyword.cos_full_notm}} credentials](#storage_class_custom) to the default storage classes, you must not list secret in the PVC.
    
    `ibm.io/endpoint`
    :   If you created your {{site.data.keyword.cos_full_notm}} service instance in a location that is different from your cluster, enter the private or public cloud service endpoint of your {{site.data.keyword.cos_full_notm}} service instance that you want to use. For more information and an overview of available service endpoints, see [Additional endpoint information](/docs/cloud-object-storage?topic=cloud-object-storage-advanced-endpoints). By default, the `ibmc` Helm plug-in automatically retrieves your cluster location and creates the storage classes by using the {{site.data.keyword.cos_full_notm}} private cloud service endpoint that matches your cluster location. If your classic cluster is in a multizone metro, such as `dal10`, the {{site.data.keyword.cos_full_notm}} private cloud service endpoint for the multizone metro, for example Dallas. To verify that the service endpoint in your storage classes matches the service endpoint of your service instance, run `oc describe storageclass < storageclassname>`. Make sure that you enter your service endpoint in the format `https://< s3fs_private_service_endpoint>` for private cloud service endpoints, or `http://< s3fs_public_service_endpoint>` for public cloud service endpoints. If the service endpoint in your storage class matches the service endpoint of your {{site.data.keyword.cos_full_notm}} service instance, don't include the `ibm.io/endpoint` option in your PVC YAML file. 
    
    `storage`
    :   In the spec resources requests section, enter a fictitious size for your {{site.data.keyword.cos_full_notm}} bucket in gigabytes. The size is required by Kubernetes, but not respected in {{site.data.keyword.cos_full_notm}}. You can enter any size that you want. The actual space that you use in {{site.data.keyword.cos_full_notm}} might be different and is billed based on the [pricing table](https://cloud.ibm.com/objectstorage/create#pricing){: external}.
    
    `storageClassName`
    :   Choose between the following options.
    :   If `ibm.io/auto-create-bucket: "true"`: Enter the storage class that you want to use for your new bucket.
    :   If `ibm.io/auto-create-bucket: "false"`: Enter the storage class that you used to create your existing bucket. 
    :   If you manually created the bucket in your {{site.data.keyword.cos_full_notm}} service instance or you can't remember the storage class that you used, find your service instance in the {{site.data.keyword.cloud_notm}} dashboard and review the **Class** and **Location** of your existing bucket. Then, use the appropriate [storage class](#cos_storageclass_reference).
        The {{site.data.keyword.cos_full_notm}} API endpoint that is set in your storage class is based on the region that your cluster is in. If you want to access a bucket that is located in a different region than the one where your cluster is in, you must create a [custom storage class](/docs/openshift?topic=openshift-kube_concepts#customized_storageclass) and use the appropriate API endpoint for your bucket.
        {: note}
        
    `secret-namespace`
    :   By default, the COS plug-in searches for your secret in the same namespace where you create the PVC. If you created your secret in a namespace other than the namespace where you want to create your PVC, enter the namespace where you created your secret.

1. Create the PVC in your cluster.

    ```sh
    oc apply -f filepath/pvc.yaml
    ```
    {: pre}

1. Verify that your PVC is created and bound to the PV.

    ```sh
    oc get pvc
    ```
    {: pre}

    Example output

    ```sh
    NAME                  STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                     AGE
    s3fs-test-pvc         Bound     pvc-b38b30f9-1234-11e8-ad2b-t910456jbe12   8Gi        RWO            ibmc-s3fs-standard-cross-region  1h
    ```
    {: screen}

1. Optional: If you plan to access your data with a non-root user, or added files to an existing {{site.data.keyword.cos_full_notm}} bucket by using the console or the API directly, make sure that the [files have the correct permission](/docs/containers?topic=containers-cos_nonroot_access) assigned so that your app can successfully read and update the files as needed.

1. To mount the PV to your deployment, create a configuration `.yaml` file and specify the PVC that binds the PV. {: #cos_app_volume_mount}

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: <deployment_name>
      labels:
        app: <deployment_label>
    spec:
      selector:
        matchLabels:
          app: <app_name>
      template:
        metadata:
          labels:
            app: <app_name>
        spec:
          containers:
          - image: <image_name>
            name: <container_name>
            securityContext:
              runAsUser: <non_root_user>
            volumeMounts:
            - name: <volume_name>
              mountPath: /<file_path>
          volumes:
          - name: <volume_name>
            persistentVolumeClaim:
              claimName: <pvc_name>
    ```
    {: codeblock}

`app`
:   In the metadata section, enter label for the deployment.

`matchLabels.app` and `labels.app`
:   In the spec selector and in the spec template metadata sections, enter a label for your app.

`image`
:   The name of the container image that you want to use. To list available images in your {{site.data.keyword.registrylong_notm}} account, run `ibmcloud cr image-list`.

`name`
:   The name of the container that you want to deploy to your cluster.

`runAsUser`
:   In the spec containers security context section, you can optionally set the run as user value.

`mountPath`
:   In the spec containers volume mounts section, enter the absolute path of the directory to where the volume is mounted inside the container. If you want to share a volume between different apps, you can specify [volume sub paths](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath){: external} for each of your apps.

`volumeMounts.name`
:   In the spec containers volume mounts section, enter the name of the volume to mount to your pod.

`volumes.name`
:   In the volumes section, enter the name of the volume to mount to your pod. Typically this name is the same as `volumeMounts/name`.

`claimName`
:   In the volumes persistent volume claim section, enter the name of the PVC that binds the PV that you want to use.


### Creating a deployment 
{: #create-cos-deployment-steps}

After you create PVC and deployment configuration files, create the deployment in your cluster.
{: shortdesc}

1. Create the deployment.

    ```sh
    oc apply -f <local_yaml_path>
    ```
    {: pre}

1. Verify that the PV is successfully mounted.

    ```sh
    oc describe deployment <deployment_name>
    ```
    {: pre}

    The mount point is in the **Volume Mounts** field and the volume is in the **Volumes** field.

    ```sh
    Volume Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-tqp61 (ro)
          /volumemount from myvol (rw)
    ...
    Volumes:
        myvol:
        Type:    PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
        ClaimName:    mypvc
        ReadOnly:    false
    ```
    {: screen}

1. Verify that you can write data to your {{site.data.keyword.cos_full_notm}} service instance by logging in to the app pod and writing data. Log in to the pod that mounts your PV.

    ```sh
    oc exec <pod_name> -it bash
    ```
    {: pre}

1. Navigate to your volume mount path that you defined in your app deployment.
1. Create a text file.

    ```sh
    echo "This is a test" > test.txt
    ```
    {: pre}

1. From the {{site.data.keyword.Bluemix}} dashboard, navigate to your {{site.data.keyword.cos_full_notm}} service instance.
1. From the menu, select **Buckets**.
1. Open your bucket, and verify that you can see the `test.txt` that you created.

    

## Using object storage in a stateful set
{: #cos_statefulset}

If you have a stateful app such as a database, you can create stateful sets that use {{site.data.keyword.cos_full_notm}} to store your app's data. Alternatively, you can use an {{site.data.keyword.cloud_notm}} database-as-a-service, such as {{site.data.keyword.cloudant_short_notm}} and store your data in the cloud.
{: shortdesc}

Before you begin:
- [Create and prepare your {{site.data.keyword.cos_full_notm}} service instance](#create_cos_service).
- [Create a secret to store your {{site.data.keyword.cos_full_notm}} service credentials](#create_cos_secret).
- [Decide on the configuration for your {{site.data.keyword.cos_full_notm}}](#configure_cos).

To deploy a stateful set that uses object storage:

1. Create a configuration file for your stateful set and the service that you use to expose the stateful set. The following examples show how to deploy NGINX as a stateful set with three replicas, each replica with a separate bucket or sharing the same bucket.

    Example to create a stateful set with three replicas, with each replica using a separate bucket.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-v01
      namespace: default
      labels:
        app: nginx-v01 # must match spec.template.metadata.labels and spec.selector.matchLabels in stateful set YAML
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx-v01 # must match spec.template.metadata.labels and spec.selector.matchLabels in stateful set YAML
    ---
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: web-v01
      namespace: default
    spec:
      selector:
        matchLabels:
          app: nginx-v01 # must match spec.template.metadata.labels in stateful set YAML and metadata.labels in service YAML
      serviceName: "nginx-v01"
      replicas: 3
      template:
        metadata:
          labels:
            app: nginx-v01 # must match spec.selector.matchLabels in stateful set YAML and metadata.labels in service YAML
        spec:
          terminationGracePeriodSeconds: 10
          containers:
          - name: nginx
            image: k8s.gcr.io/nginx-slim:0.8
            ports:
            - containerPort: 80
              name: web
            volumeMounts:
            - name: mypvc
              mountPath: /usr/share/nginx/html
      volumeClaimTemplates:
      - metadata:
          name: mypvc
          annotations:
            ibm.io/auto-create-bucket: "true"
            ibm.io/auto-delete-bucket: "true"
            ibm.io/bucket: ""
            ibm.io/secret-name: mysecret
            volume.beta.kubernetes.io/storage-class: ibmc-s3fs-standard-perf-cross-region
            volume.beta.kubernetes.io/storage-provisioner: ibm.io/ibmc-s3fs
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "ibmc-s3fs-standard-perf-cross-region"
          resources:
            requests:
              storage: 1Gi
    ```
    {: codeblock}

    Example to create a stateful set with three replicas that share the same bucket `mybucket`.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-v01
      namespace: default
      labels:
        app: nginx-v01 # must match spec.template.metadata.labels and spec.selector.matchLabels in stateful set YAML
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx-v01 # must match spec.template.metadata.labels and spec.selector.matchLabels in stateful set YAML
    ---
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: web-v01
      namespace: default
    spec:
      selector:
        matchLabels:
          app: nginx-v01 # must match spec.template.metadata.labels in stateful set YAML and metadata.labels in service YAML
      serviceName: "nginx-v01"
      replicas: 3
      template:
        metadata:
          labels:
            app: nginx-v01 # must match spec.selector.matchLabels in stateful set YAML and metadata.labels in service YAML
        spec:
          terminationGracePeriodSeconds: 10
          containers:
          - name: nginx
            image: k8s.gcr.io/nginx-slim:0.8
            ports:
            - containerPort: 80
              name: web
            volumeMounts:
            - name: mypvc
              mountPath: /usr/share/nginx/html
      volumeClaimTemplates:
      - metadata:
          name: mypvc
          annotations:
            ibm.io/auto-create-bucket: "false"
            ibm.io/auto-delete-bucket: "false"
            ibm.io/bucket: mybucket
            ibm.io/secret-name: mysecret
            volume.beta.kubernetes.io/storage-class: ibmc-s3fs-standard-perf-cross-region
            volume.beta.kubernetes.io/storage-provisioner: ibm.io/ibmc-s3fs
        spec:
          accessModes: [ "ReadOnlyMany" ]
          storageClassName: "ibmc-s3fs-standard-perf-cross-region"
          resources:
            requests:
              storage: 1Gi
    ```
    {: codeblock}


`name`
:   Enter a name for your stateful set. The name that you enter is used to create the name for your PVC in the format: `<volume_name>-< statefulset_name>-<replica_number>`.

`serviceName`
:   Enter the name of the service that you want to use to expose your stateful set.

`replicas`
:   Enter the number of replicas for your stateful set.

`matchLabels`
:   In the spec selector match labels section, enter all labels that you want to include in your stateful set and your PVC. Labels that you include in the `volumeClaimTemplates` of your stateful set are not recognized by Kubernetes. Instead, you must define these labels in the `spec.selector.matchLabels` and `spec.template.metadata.labels` section of your stateful set YAML. To make sure that all your stateful set replicas are into the load balancing of your service, include the same label that you used in the `spec.selector` section of your service YAML.

`labels`
:   In the spec metadata labels section, enter the same labels that you added to the `spec.selector.matchLabels` section of your stateful set YAML.

`terminationGracePeriodSeconds`
:   Enter the number of seconds to give the `kubelet` to gracefully terminate the pod that runs your stateful set replica. For more information, see [Delete Pods](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/#delete-pods).

`VolumeClaimTemplates.name`
:   In the spec volume claim templates metadata section, enter a name for your volume. Use the same name that you defined in the `spec.containers.volumeMount.name` section. The name that you enter here is used to create the name for your PVC in the format: `<volume_name>-<statefulset_name>-<replica_number>`.

`ibm.io/auto-create-bucket`
:   In the spec volume claim templates metadata section, set an annotation to configure how buckets are created. Choose between the following options: 
    - **true:** Choose this option to automatically create a bucket for each stateful set replica. Note that the service credentials must have **Writer** permissions to automatically create the bucket.
    - **false:** Choose this option if you want to share an existing bucket across your stateful set replicas. Make sure to define the name of the bucket in the `spec.volumeClaimTemplates.metadata.annotions.ibm.io/bucket` section of your stateful set YAML.

`ibm.io/auto-delete-bucket`
:   In the spec volume claim templates metadata section, set an annotation to configure how buckets are deleted. Choose between the following options: 
    - **true:** Your data, the bucket, and the PV is automatically removed when you delete the PVC. Your {{site.data.keyword.cos_full_notm}} service instance remains and is not deleted. If you choose to set this option to true, then you must set `ibm.io/auto-create-bucket: true` and `ibm.io/bucket: ""`so that your bucket is automatically created with a name with the format `tmp-s3fs-xxxx`.
    - **false:** When you delete the PVC, the PV is deleted automatically, but your data and the bucket in your {{site.data.keyword.cos_full_notm}} service instance remain. To access your data, you must create a new PVC with the name of your existing bucket.

`ibm.io/bucket`
:   In the spec volume claim templates metadata section, set an annotation for the bucket details. Choose between the following options:
    - If `ibm.io/auto-create-bucket` is set to **true**: Enter the name of the bucket that you want to create in {{site.data.keyword.cos_full_notm}}. If in addition `ibm.io/auto-delete-bucket` is set to `true`, you must leave this field blank to automatically assign your bucket a name with the format `tmp-s3fs-xxxx`. The name must be unique in {{site.data.keyword.cos_full_notm}}.
    - If `ibm.io/auto-create-bucket` is set to **false**: Enter the name of the existing bucket that you want to access in the cluster.

`ibm.io/secret-name`
:   In the spec volume claim templates metadata annotations section, enter the name of the secret that holds the {{site.data.keyword.cos_full_notm}} credentials that you created earlier. If you add your [{{site.data.keyword.cos_full_notm}} credentials](#storage_class_custom) to the default storage classes, you must not list the secret in the PVC.

`kubernetes.io/storage-class`
:   In the spec volume claim templates metadata annotations section, enter the storage class that you want to use. Choose between the following options:
    - If `ibm.io/auto-create-bucket: "true"`: Enter the storage class that you want to use for your new bucket.
    - If `ibm.io/auto-create-bucket: "false"`: Enter the storage class that you used to create your existing bucket.
    To list existing storage classes, run`oc get storageclasses | grep s3`. If you don't specify a storage class, the PVC is created with the default storage class that is set in your cluster. Make sure that the default storage class uses the `ibm.io/ibmc-s3fs` provisioner so that your stateful set is provisioned with object storage.

`storageClassName`
:   In the spec volume claim templates spec section, enter the same storage class that you entered in the `spec.volumeClaimTemplates.metadata.annotations.volume.beta.kubernetes.io/storage-class` section of your stateful set YAML.

`storage`
:   In the spec volume claim templates spec resource requests section, enter a fictitious size for your {{site.data.keyword.cos_full_notm}} bucket in gigabytes. The size is required by Kubernetes, but not respected in {{site.data.keyword.cos_full_notm}}. You can enter any size that you want. The actual space that you use in {{site.data.keyword.cos_full_notm}} might be different and is billed based on the [pricing table](https://cloud.ibm.com/objectstorage/create#pricing){: external}.



## Backing up and restoring data
{: #cos_backup_restore}

{{site.data.keyword.cos_full_notm}} is set up to provide high durability for your data so that your data is protected from being lost. You can find the SLA in the [{{site.data.keyword.cos_full_notm}} service terms](http://www.ibm.com/support/customer/csol/terms/?id=i126-7857&lc=en){: external}.
{: shortdesc}

{{site.data.keyword.cos_full_notm}} doesn't provide a version history for your data. If you need to maintain and access older versions of your data, you must set up your app to manage the history of data or implement alternative backup solutions. For example, you might want to store your {{site.data.keyword.cos_full_notm}} data in your on-prem database or use tapes to archive your data.
{: note}



## Adding your {{site.data.keyword.cos_full_notm}} credentials to the default storage classes
{: #storage_class_custom}

You can create a [Kubernetes secret with your {{site.data.keyword.cos_full_notm}} Administrator credentials](#create_cos_secret) and list the secret in your storage classes. By adding your {{site.data.keyword.cos_full_notm}} credentials to the default storage classes, users in your cluster don't need to provide COS credentials in their PVCs. Note that storage classes are available across namespaces.

Before you begin:
* [Access your {{site.data.keyword.openshiftshort}} cluster](/docs/openshift?topic=openshift-access_cluster).
* Have the **Manager** service access role for the cluster.
* Have the **Writer** service access role for the {{site.data.keyword.cos_full_notm}} service instance.

To add a secret to a storage class:

1. Create a [Kubernetes secret with your {{site.data.keyword.cos_full_notm}} credentials](#create_cos_secret). Note that your secret and app pods must be in the same namespace. Note that the service credentials must have at least **Writer** permission to use the `auto-create-bucket` dynamic provisioning feature.

2. List the default storage classes.
    ```sh
    oc get sc | grep s3fs
    ```
    {: pre}

3. Export the details for the default storage class that you want to edit in `YAML` form.

    * Example command for the `ibmc-s3fs-flex-regional` storage class.
        ```sh
        oc get sc ibmc-s3fs-flex-regional -o yaml > ibmc-s3fs-flex-regional.yaml
        ```
        {: pre}
        
    * Generic command for exporting a storage class.
        ```sh
        oc get sc <storage-class> -o yaml > <storage-class>.yaml 
        ```
        {: pre}

4. Add the `ibm.io/secret-name: "<secret_name>"` parameter to the `parameters` field of the storage class.
    ```yaml
    ...
    parameters:
        ibm.io/secret-name: "<secret_name>" # Enter the name the secret that you created.
      ...
    ```
    {: codeblock}
    
5. Delete the original storage class from your cluster.
    ```sh
    oc delete sc <storage-class>
    ```
    {: pre}

6. Create the storage class in your cluster.
    ```sh
    oc apply -f <storage_class.yaml>
    ```
    {: pre}

7. Create a PVC that refers to the storage class that you customized. Note that you don't need to provide secret details in your PVC.
    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: <pvc-name>
    namespace: <namespace>
      annotations:
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
      storageClassName: ibmc-s3fs-flex-regional
      volumeMode: Filesystem
    ```
    {: codeblock}

8. Repeat these steps for any of the default storage classes that you want customize.


## Storage class reference
{: #cos_storageclass_reference}

### Standard
{: #standard}



Name
:   `ibmc-s3fs-standard-cross-region`
:   `ibmc-s3fs-standard-perf-cross-region`
:   `ibmc-s3fs-standard-regional`
:   `ibmc-s3fs-standard-perf-regional`


Default resiliency endpoint
:   The resiliency endpoint is automatically set based on the location that your cluster is in. For more information, see [Regions and endpoints](/docs/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#endpoints).

Chunk size
:   Storage classes without `perf`: 16 MB. Storage classes with `perf`: 52 MB


Kernel cache
:   Enabled

Billing
:   Monthly

Pricing
:   Review the [pricing documentation](https://cloud.ibm.com/objectstorage/create#pricing){: external}.


### Vault
{: #Vault}



Name
:   `ibmc-s3fs-vault-cross-region`
:   `ibmc-s3fs-vault-regional`

Default resiliency endpoint
:   The resiliency endpoint is automatically set based on the location that your cluster is in. For more information, see [Regions and endpoints](/docs/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#endpoints).

Chunk size
:   16 MB

Kernel cache
:   Disabled

Billing
:   Monthly

Pricing
:   Review the [pricing documentation](https://cloud.ibm.com/objectstorage/create#pricing){: external}.

### Cold
{: #cold}


Name
:   `ibmc-s3fs-cold-cross-region`
:   `ibmc-s3fs-cold-perf-cross-region`
:   `ibmc-s3fs-cold-regional`
:   `ibmc-s3fs-cold-perf-regional`

Default resiliency endpoint
:   The resiliency endpoint is automatically set based on the location that your cluster is in. For more information, see [Regions and endpoints](/docs/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#endpoints).

Chunk size
:   16 MB

Kernel cache
:   Disabled

Billing
:   Monthly

Pricing
:   Review the [pricing documentation](https://cloud.ibm.com/objectstorage/create#pricing){: external}.

### Flex
{: #flex}


Name
:   `ibmc-s3fs-flex-cross-region`
:   `ibmc-s3fs-flex-perf-cross-region`
:   `ibmc-s3fs-flex-regional`
:   `ibmc-s3fs-flex-perf-regional`

Default resiliency endpoint
:   The resiliency endpoint is automatically set based on the location that your cluster is in. For more information, see [Regions and endpoints](/docs/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#endpoints).

Chunk size
:   Storage classes without `perf`: 16 MB
:   Storage classes with `perf`: 52 MB.

Kernel cache
:   Disabled

Billing
:   Monthly

Pricing
:  Review the [pricing documentation](https://cloud.ibm.com/objectstorage/create#pricing){: external}.

## Limitations
{: #cos_limitations}

* {{site.data.keyword.cos_full_notm}} is based on the `s3fs-fuse` file system. You can review a list of limitations in the [`s3fs-fuse` repository](https://github.com/s3fs-fuse/s3fs-fuse#limitations).
* To access a file in {{site.data.keyword.cos_full_notm}} with a non-root user, you must set the `runAsUser` and `fsGroup` values in your deployment to the same value.



