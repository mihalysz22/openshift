---

copyright: 
  years: 2014, 2022
lastupdated: "2022-03-08"

keywords: openshift

subcollection: openshift

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}


# Debugging app deployments
{: #debug_apps}
{: support}

Review the options that you have to debug your app deployments and find the root causes for failures.
{: shortdesc}

**Infrastructure provider**:
* ![Classic infrastructure provider icon.](images/icon-classic-2.svg) Classic
* ![VPC infrastructure provider icon.](images/icon-vpc-2.svg) VPC

Before you begin, ensure you have the [**Writer** or **Manager** {{site.data.keyword.cloud_notm}} IAM service access role](/docs/openshift?topic=openshift-users#checking-perms) for the namespace where your app is deployed.



1. Make sure that you review the [common scenarios where you might need to modify your apps](/docs/openshift?topic=openshift-plan_deploy#openshift_move_apps_scenarios) so that you can deploy them on {{site.data.keyword.redhat_openshift_notm}} clusters.

1. Look for abnormalities in the service or deployment resources by running the `describe` command.
    ```sh
    oc describe service <service_name>
    ```
    {: pre}

2. [Check whether the containers are stuck in the `ContainerCreating` state](/docs/containers?topic=containers-readonly_nodes).

3. Check whether the cluster is in the `Critical` state. If the cluster is in a `Critical` state, check the firewall rules and verify that the master can communicate with the worker nodes.

4. Verify that the service is listening on the correct port.
    1. Get the name of a pod.
        ```sh
        oc get pods
        ```
        {: pre}

    2. Log in to a container.
        ```sh
        oc exec -it <pod_name> -- /bin/bash
        ```
        {: pre}

    3. Curl the app from within the container. If the port is not accessible, the service might not be listening on the correct port or the app might have issues. Update the configuration file for the service with the correct port and redeploy or investigate potential issues with the app.
        ```sh
        curl localhost: <port>
        ```
        {: pre}

5. Verify that the service is linked correctly to the pods.
    1. Get the name of a pod.
        ```sh
        oc get pods
        ```
        {: pre}

    2. Log in to a container.
        ```sh
        oc exec -it <pod_name> -- /bin/bash
        ```
        {: pre}

    3. Curl the cluster IP address and port of the service.
        ```sh
        curl <cluster_IP>:<port>
        ```
        {: pre}

    4. If the IP address and port are not accessible, look at the endpoints for the service.
        * If no endpoints are listed, then the selector for the service does not match the pods. For example, your app deployment might have the label `app=foo`, but the service might have the selector `run=foo`.
        * If endpoints are listed, then look at the target port field on the service and make sure that the target port is the same as what is being used for the pods. For example, your app might listen on port 9080, but the service might listen on port 80.

6. For Ingress services, verify that the service is accessible from within the cluster.
    1. Get the name of a pod.
        ```sh
        oc get pods
        ```
        {: pre}

    2. Log in to a container.
        ```sh
        oc exec -it <pod_name> -- /bin/bash
        ```
        {: pre}

    2. Curl the URL specified for the Ingress service. If the URL is not accessible, check for a firewall issue between the cluster and the external endpoint.
        ```sh
        curl <host_name>.<domain>
        ```
        {: pre}








