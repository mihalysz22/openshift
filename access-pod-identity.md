---

copyright: 
  years: 2014, 2022
lastupdated: "2022-03-08"

keywords: openshift

subcollection: openshift


---

{{site.data.keyword.attribute-definition-list}}


# Authorizing pods in your cluster to {{site.data.keyword.cloud_notm}} services with IAM trusted profiles
{: #pod-iam-identity}

Give application pods that run in your {{site.data.keyword.openshiftlong}} cluster access to {{site.data.keyword.cloud_notm}} services by creating a trusted profile in {{site.data.keyword.cloud_notm}} Identity and Access Management (IAM).  
{: shortdesc}

Authorizing pods with IAM trusted profiles is available for clusters that run {{site.data.keyword.redhat_openshift_notm}} version 4.7 and later.
{: note}

- Clusters that are running version 4.7 or later and were created after July 2021: Authorizing pods with IAM trusted profiles is enabled automatically.
- Clusters that are running version 4.7 or later and were created before July 2021: You can enable IAM trusted profiles by running the [`ibmcloud oc cluster master refresh`](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_apiserver_refresh) command. Note that if you have refreshed your cluster master after July 2021, IAM trusted profiles are enabled.

In IAM
:   Start by creating an IAM trusted profile. Then, link the trusted profile with your {{site.data.keyword.openshiftlong_notm}} compute resource by selecting conditions to match with your clusters, including a Kubernetes namespace and service account in the clusters. Finally, assign access policies to the {{site.data.keyword.cloud_notm}} services that you want your apps to use.

In your cluster
:   Through [Kubernetes service account token volume projection](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-token-volume-projection){: external}, the apps that run in your linked cluster's [Kubernetes namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/){: external} and use the namespace's service account can exchange the service account public key to get an {{site.data.keyword.cloud_notm}} IAM access token. Your app can use this access token to authenticate API requests to {{site.data.keyword.cloud_notm}} services, such as databases, {{site.data.keyword.watson}}, or VPC infrastructure. Through the access policies of the trusted profile, you control what actions the token lets the app perform.

## Prerequisites
{: #iam-identity-prereqs}

To use {{site.data.keyword.cloud_notm}} IAM identities for pods in your cluster, verify that you meet the prerequisites.
{: shortdesc}

Supported infrastructure providers

- ![VPC infrastructure provider icon.](images/icon-vpc-2.svg) VPC  
- ![Classic infrastructure provider icon.](images/icon-classic-2.svg) Classic  
- ![Satellite infrastructure provider icon.](images/icon-satellite.svg) {{site.data.keyword.satelliteshort}}


Minimum required permissions
- **Viewer** [platform](/docs/openshift?topic=openshift-access_reference#iam_platform) access role and the **Writer** [service](/docs/openshift?topic=openshift-access_reference#service) access role for the cluster in {{site.data.keyword.cloud_notm}} IAM for **{{site.data.keyword.containershort}}**.
- The `iam-identity.profile.create` and `iam-identity.profile.linkToResource` actions for the [IAM identity service](/docs/account?topic=account-iam-service-roles-actions#iam-identity-service).

Supported versions

- The cluster must be [created](/docs/containers?topic=containers-clusters) at {{site.data.keyword.redhat_openshift_notm}} version 4.7 or later.
- For new clusters, authorizing pods with IAM trusted profiles is enabled automatically. You can enable IAM trusted profiles on existing clusters by running [`ibmcloud oc cluster master refresh`](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_apiserver_refresh).



## Creating an IAM trusted profile for your cluster in the API
{: #iam-identity-create-api}
{: api}

As an account administrator, create a trusted profile in {{site.data.keyword.cloud_notm}} Identity and Access Management. For more information, see the [IAM documentation](/docs/account?topic=account-create-trusted-profile&interface=ui).
{: shortdesc}

**Before you begin**: Verify that you meet the [prerequisites](#iam-identity-prereqs).

[Access your {{site.data.keyword.redhat_openshift_notm}} cluster](/docs/openshift?topic=openshift-access_cluster).

1. Create an [IAM trusted profile](/docs/account?topic=account-create-trusted-profile) in {{site.data.keyword.cloud_notm}} Identity and Access Management. Note the `uuid` (profile ID) in the output.

    * Replace `<access_token>` with your {{site.data.keyword.cloud_notm}} token. To get this token from the command line, log in and run `ibmcloud iam oauth-tokens`.
    * Enter a `<profile_name>` and optional `<description>` for the IAM trusted profile.
    * Replace `<account_id>` with the ID of your {{site.data.keyword.cloud_notm}} account. To get this ID from the command line, run `ibmcloud account show`.

        ```sh
        curl -L -X POST 'https://iam.cloud.ibm.com/v1/profiles' \
        -H 'Accept: application/json' \
        -H 'Authorization: Bearer <access_token>' \
        -H 'Content-Type: application/json' \
        --data-raw '{
            "name": "<profile_name>",
            "description": "<description>",
            "account_id": "<account_id>"
        }'
        ```
        {: codeblock}

2. Link the IAM trusted profile to a Kubernetes namespace in your {{site.data.keyword.openshiftlong_notm}} cluster.
    1. Get your `<profile-id>` from the output of the first step. Or, you can run the following API command.
        ```sh
        curl -L -X GET 'https://iam.cloud.ibm.com/v1/profiles/?account_id=<account_id>' \
        -H 'Accept: application/json' \
        -H 'Authorization: <access_token>' \
        -H 'Content-Type: application/json' \
        ```
        {: codeblock}

    2. Get your cluster name and CRN by running the following API command.
        ```sh
        curl --location --request GET 'https://containers.cloud.ibm.com/global/v1/clusters' \
        --header 'Authorization: <access_token>'
        ```
        {: codeblock}

    3. Link the IAM trusted profile to your cluster.

        * For `<link_name>`, enter a name for the link between the trusted profile and the cluster.
        * Replace `<profile-id>`, `<access_token>`, `<cluster_crn>`, and `<cluster_name>` with the values that you previously retrieved.
        * For `<ns>`, enter the namespace in your cluster. You can list namespaces by logging in to the cluster and running `oc get ns`. The Kubernetes namespace that you enter does not have to exist already. Any future namespace with this name can establish trust.

            ```sh
            curl -L -X POST 'https://iam.cloud.ibm.com/v1/profiles/<profile-id>/links' \
            -H 'Accept: application/json' \
            -H 'Authorization: Bearer <access_token>' \
            -H 'Content-Type: application/json' \
            --data-raw '{
                "name": "<link_name>",
                "cr_type": "ROKS_SA",
                "link": {
                    "crn": "<cluster_crn>",
                    "namespace": "<ns>",
                    "name": "<cluster_name>"
                }
            }'
            ```
            {: codeblock}

3. [Assign the trusted profile to an access group](/docs/account?topic=account-groups#access_ag_api) with access policies to the {{site.data.keyword.cloud_notm}} services that you want your apps to have access to.

Your clusters can now establish trust with {{site.data.keyword.cloud_notm}} IAM. Next, your developers can [configure their application pods to exchange the identity token for an IAM token to authenticate requests to {{site.data.keyword.cloud_notm}} services](#iam-identity-pod).


## Creating an IAM trusted profile for your cluster in the UI
{: #iam-identity-create-ui}
{: ui}

As an account administrator, create a trusted profile in {{site.data.keyword.cloud_notm}} Identity and Access Management. For more information, see the [IAM documentation](/docs/account?topic=account-create-trusted-profile&interface=ui).
{: shortdesc}

**Before you begin**: Verify that you meet the [prerequisites](#iam-identity-prereqs).

1. Log in to the [{{site.data.keyword.cloud_notm}} IAM **Trusted profiles** console](https://cloud.ibm.com/iam/trusted-profiles){: external}.
2. Click **Create**.
3. Give your trusted profile a name and click **Continue**.
4. For trust entity type, select **Compute resources**.
5. For compute service, select **Kubernetes**.
6. For compute resources, choose among the following options to determine which clusters or resources in the cluster can establish trust.
    * **All service resources**: Any existing and future clusters. To link the trusted profile only to clusters under certain conditions, you can **Add new condition +**. For example, you might add conditions such as only clusters in a resource group, only clusters with a `prod` Kubernetes namespace, or only clusters with a `trusted-profile` Kubernetes service account.
    * **Specific resources**: A particular cluster. Click **Add another resource +** to select a cluster and fill out conditions such as the Kubernetes namespace or Kubernetes service account that can establish trust.
    * Click **Continue**.

    The Kubernetes namespace and service account names that you enter don't have to exist already. Any future namespaces or service accounts with these names can establish trust. To list existing namespaces, log in to your cluster and run `oc get ns`. To list existing service accounts, log in to your cluster and run `oc get sa -n <namespace>`.
    {: note}

7. Grant the trusted profile any access policies to the **IAM services** and **Account management** that you want the pod in your cluster to be able to access.
    * For example, you might grant access to **All Identity and Access enabled services** and **All resources** in those services, with the **Editor** platform, **Writer** service, and **Viewer** resource group access roles.
    * Click **Add +** to add the policy to the profile, and continue adding as many policies as you want.
8. In the **Summary** pane, review the profile, trust, and access details. Then, click **Create**.

Your clusters can now establish trust with {{site.data.keyword.cloud_notm}} IAM. Next, your developers can [configure their application pods to exchange the identity token for an IAM token to authenticate requests to {{site.data.keyword.cloud_notm}} services](#iam-identity-pod).


## Configure your application pods to authenticate with {{site.data.keyword.cloud_notm}} services
{: #iam-identity-pod}

As a developer, you can configure your application pods to authenticate with {{site.data.keyword.cloud_notm}} services in clusters that are linked to an IAM trusted profile set up.
{: shortdesc}

Before you begin:
* [Access your {{site.data.keyword.redhat_openshift_notm}} cluster](/docs/openshift?topic=openshift-access_cluster).
* Make sure that your account administrator [created an IAM trusted profile for your cluster](#iam-identity-create-api).

To configure your application pods to authenticate with {{site.data.keyword.cloud_notm}} services:

1. [Design your pod configuration file](/docs/containers?topic=containers-app) to use [service account token volume projection](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-token-volume-projection){: external}. 
    1. In the `containers` section, mount the identity token in the `volumeMounts` section.
        ```yaml
        ...
            volumeMounts:
            - mountPath: /var/run/secrets/tokens
              name: sa-token
        ```
        {: codeblock}

    2. In the `volumes` section, set up the service account token volume projection.

        Modify the `expirationSeconds` field to control how long the token is valid for. To retrieve IAM tokens, the service account token expiration must be 1 hour or less.
        {: tip}

        ```yaml
        ...
        volumes:
          - name: sa-token
            projected:
              sources:
              - serviceAccountToken:
                  path: sa-token
                  expirationSeconds: 3600
                  audience: iam
        ...
        ```
        {: codeblock}

2. Design your app to exchange the service account projected token for an IAM token that you can use for subsequent API calls to {{site.data.keyword.cloud_notm}} services. Review the following example authentication request. Replace `${profile_id}`: Replace with the ID of the trusted profile that the cluster is linked to. To list available profile IDs, you or the account administrator can use the `ic iam tps` command, the `GET 'https://iam.cloud.ibm.com/v1/profiles/?account_id=<account_id>'`, or you can view the trusted profiles in the [IAM console](https://cloud.ibm.com/iam/trusted-profiles/){: external}.

    ```sh
    curl -s -X POST \
        -H \"Content-Type: application/x-www-form-urlencoded\" \
        -H \"Accept: application/json\" \
        -d grant_type=urn:ibm:params:oauth:grant-type:cr-token \
        -d cr_token=\$(cat /var/run/secrets/tokens/sa-token) \
        -d profile_id=${profile_id} \
        https://iam.cloud.ibm.com/identity/token
    ```
    {: codeblock}

3. Before you deploy your app, try an example Kubernetes job to test the token exchange. In the following Kubernetes job, a `curl` pod makes an API request to {{site.data.keyword.cloud_notm}} IAM to verify that the cluster's public key is exchanged for an IAM access token. Your app might call other {{site.data.keyword.cloud_notm}} services that the trusted profile authorizes.

    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: token-exchange-job
      namespace: default
    spec:
      template:
        spec:
          containers:
          - name: curl
            image: curlimages/curl:7.77.0
            command: ["/bin/sh"]
            args: ["-c", "curl -s -H \"Content-Type: application/x-www-form-urlencoded\" -H \"Accept: application/json\" -d grant_type=urn:ibm:params:oauth:grant-type:cr-token -d cr_token=$(cat /var/run/secrets/tokens/sa-token) -d profile_id==<profile_id> https://iam.cloud.ibm.com/identity/token"]
            volumeMounts:
            - mountPath: /var/run/secrets/tokens
              name: sa-token
          restartPolicy: Never
          serviceAccountName: default
          volumes:
          - name: sa-token
            projected:
              sources:
              - serviceAccountToken:
                  path: sa-token
                  expirationSeconds: 3600
                  audience: iam
    ```
    {: codeblock}

4. Use the exchanged token to authenticate subsequent requests to {{site.data.keyword.cloud_notm}} services, such as by storing the token as a variable that is reused in the rest of your code.
5. [Deploy your app](/docs/containers?topic=containers-deploy_app).
6. Check that your pod is running.
    ```sh
    oc get pods -n <namespace>
    ```
    {: pre}

7. Check the logs of your pod to verify that an `access_token` is successfully returned. If not, look for an `errorCode` with troubleshooting information.
    ```sh
    oc logs <pod>
    ```
    {: pre}

8. Optional: Log in to the pod and verify that you get a successful API request from an {{site.data.keyword.cloud_notm}} service by using the `$TOKEN` that you retrieved in the logs. The following example returns a list of classic clusters and requires the **Viewer** platform access role to **Kubernetes Service**.
    ```sh
    curl -X GET https://containers.cloud.ibm.com/global/v2/classic/getClusters -H "Authorization: Bearer $TOKEN"
    ```
    {: pre}


