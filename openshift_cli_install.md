---

copyright:
  years: 2014, 2022
lastupdated: "2022-03-01"

keywords: openshift, oc

subcollection: openshift

---


{{site.data.keyword.attribute-definition-list}}


# Installing the OpenShift CLI
{: #openshift-cli}

You can use the {{site.data.keyword.openshiftlong}} command line interface (CLI) plug-in (`ibmcloud oc`) to create and manage your {{site.data.keyword.redhat_openshift_notm}} cluster infrastructure, such as creating clusters and worker nodes. Then, you can use the {{site.data.keyword.redhat_openshift_notm}} CLI (`oc`) to manage the resources within your {{site.data.keyword.redhat_openshift_notm}} cluster, such as projects, pods, and deployments. To use the API, see [Setting up the API](/docs/openshift?topic=openshift-cs_api_install).

## Installing the IBM Cloud CLI and plug-ins
{: #cs_cli_install_steps}

Install the required CLIs to create and manage your {{site.data.keyword.redhat_openshift_notm}} clusters in {{site.data.keyword.openshiftlong_notm}}, and to deploy containerized apps to your cluster.
{: shortdesc}

This task includes the information for installing these CLIs and plug-ins:

* {{site.data.keyword.cloud_notm}} CLI (`ibmcloud`)
* {{site.data.keyword.openshiftlong_notm}} plug-in (`ibmcloud oc` alias for {{site.data.keyword.redhat_openshift_notm}} clusters)
* {{site.data.keyword.registrylong_notm}} plug-in (`ibmcloud cr`)
* {{site.data.keyword.containerlong_notm}} observability plug-in (`ibmcloud ob`)

If you want to use the {{site.data.keyword.cloud_notm}} console instead, you can run CLI commands directly from your web browser in the [{{site.data.keyword.cloud-shell_notm}}](#cloud-shell).
{: tip}

To install the CLIs:

1. Install the stand-alone [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cli-install-ibmcloud-cli) (`ibmcloud`).

    Plan to use the CLI often? Try [Enabling autocompletion for the {{site.data.keyword.cloud_notm}} CLI (Linux/macOS only)](/docs/cli/reference/ibmcloud?topic=cli-shell-autocomplete#shell-autocomplete-linux).
    {: tip}

2. Log in to the {{site.data.keyword.cloud_notm}} CLI. Enter your {{site.data.keyword.cloud_notm}} credentials when prompted.
    ```sh
    ibmcloud login
    ```
    {: pre}

    If you have a federated ID, use `ibmcloud login --sso` to log in to the {{site.data.keyword.cloud_notm}} CLI. Enter your username and use the provided URL in your CLI output to retrieve your one-time passcode. You know you have a federated ID when the login fails without the `--sso` and succeeds with the `--sso` option.
    {: tip}

4. Install the {{site.data.keyword.cloud_notm}} plug-in for {{site.data.keyword.openshiftlong_notm}} (`ibmcloud oc`). Use this plug-in to create and manage {{site.data.keyword.openshiftlong_notm}} resources such as clusters, worker nodes, or network load balancers.
    ```sh
    ibmcloud plugin install container-service
    ```
    {: pre}

5. Install the {{site.data.keyword.cloud_notm}} plug-in for {{site.data.keyword.registrylong_notm}} (`ibmcloud cr`). Use this plug-in to set up your own namespace in a multi-tenant, highly available, and scalable private image registry that is hosted by IBM, and to store and share Docker images with other users. Docker images are required to deploy containers into a cluster.
    ```sh
    ibmcloud plugin install container-registry
    ```
    {: pre}

6. To create a logging configuration for {{site.data.keyword.la_full_notm}} or a monitoring configuration for {{site.data.keyword.mon_full_notm}} for your cluster, install the {{site.data.keyword.containerlong_notm}} observability plug-in (`ibmcloud ob`).
    ```sh
    ibmcloud plugin install observe-service
    ```
    {: pre}

7. Verify that the plug-ins are installed correctly.
    ```sh
    ibmcloud plugin list
    ```
    {: pre}

    Example output

    ```sh
    Listing installed plug-ins...

    Plugin Name                            Version   Status
    container-registry                     0.1.514
    container-service/kubernetes-service   1.0.231
    observe-service/ob                     1.0.61
    ```
    {: screen}

For reference information about these CLIs, see the documentation for those tools.

-   [`ibmcloud` commands](/docs/cli/reference/ibmcloud?topic=cli-ibmcloud_cli#ibmcloud_cli)
-   [`ibmcloud oc` commands](/docs/openshift?topic=openshift-kubernetes-service-cli)
-   [`ibmcloud cr` commands](/docs/Registry?topic=container-registry-cli-plugin-containerregcli)






## Installing the OpenShift Origin CLI (`oc`)
{: #cli_oc}
{: support}
{: help}

To view a local version of the {{site.data.keyword.redhat_openshift_notm}} dashboard and to deploy apps into your {{site.data.keyword.openshiftlong_notm}} clusters, install the {{site.data.keyword.redhat_openshift_notm}} CLI (`oc`) and Kubernetes CLI (`kubectl`).
{: shortdesc}

Using both community Kubernetes and {{site.data.keyword.redhat_openshift_notm}} clusters? Your clusters might run different versions of Kubernetes, such as 1.11 on {{site.data.keyword.redhat_openshift_notm}} and 1.22.7 on Ubuntu. Make sure to use the `kubectl` binary file that matches the `+/- 1` [skew policy](https://kubernetes.io/releases/version-skew-policy/){: external} for your cluster `major.minor` {{site.data.keyword.redhat_openshift_notm}} and Kubernetes versions. For supported versions, see [{{site.data.keyword.redhat_openshift_notm}} versions](/docs/openshift?topic=openshift-openshift_versions#version_types).
{: note}

Looking for a simple installation? You can try installing the plug-in with a package manager such as Homebrew. Keep in mind that the version skew might not match your cluster. For example, in Homebrew, you can update Homebrew by running `brew update` and check what version of the plug-in is installed by running `brew info openshift-cli`. Then, install the plug-in by running `brew install openshift-cli`.
{: tip}

1. Download the latest {{site.data.keyword.redhat_openshift_notm}} CLI (`oc`) for your local operating system and {{site.data.keyword.redhat_openshift_notm}} version. The current default {{site.data.keyword.redhat_openshift_notm}} version is 4.8. If you use Windows, install the `oc` CLI in the same directory as the {{site.data.keyword.cloud_notm}} CLI. This setup saves you some file path changes when you run commands later. You can get the download link for `oc` from the {{site.data.keyword.redhat_openshift_notm}} web console, or manually from the `mirror.openshift.com` website.

    **![Version 4 icon.](images/icon-version-43.png) Version 4 clusters only: Download the `oc` CLI from your cluster.**
    1. From the [{{site.data.keyword.redhat_openshift_notm}} clusters console](https://cloud.ibm.com/kubernetes/clusters?platformType=openshift){: external}, select your cluster.
    2. Click **{{site.data.keyword.redhat_openshift_notm}} web console**.
    3. From the menu bar, click the information (question mark) icon.
    4. Click **Command Line Tools**.
    5. Click the download link for your operating system.

    **![Version 3.11 icon.](images/icon-version-311.png) ![Version 4 icon.](images/icon-version-43.png) Version 3 or 4 clusters: Links to manually download the `oc` CLI from `mirror.openshift.com`.**
    * [OpenShift Container Platform version 3 `oc` download link](https://mirror.openshift.com/pub/openshift-v3/clients/){: external}
    * [OpenShift Container Platform version 4 `oc` download links](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/){: external}.

2. Download the Kubernetes CLI (`kubectl`) for your {{site.data.keyword.redhat_openshift_notm}} cluster. You might use different `kubectl` versions if you have community Kubernetes clusters that run other Kubernetes versions such as 1.23.3.

    * ![Version 3.11 icon.](images/icon-version-311.png) For clusters that run {{site.data.keyword.redhat_openshift_notm}} 3.11: Download at least version 1.15. Because earlier `kubectl` versions are no longer supported, the `kubectl` version that you use for your {{site.data.keyword.redhat_openshift_notm}} 3.11, Kubernetes 1.11 cluster can't meet the `+/- 1` [skew policy](https://kubernetes.io/releases/version-skew-policy/){: external}. Instead, use at least `kubectl` version 1.15 to protect your cluster from Common Vulnerability and Exposures (CVEs) that might arise in unsupported versions.
        - **OS X**: [https://storage.googleapis.com/kubernetes-release/release/v1.15.11/bin/darwin/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.15.11/bin/darwin/amd64/kubectl){: external}
        - **Linux**: [https://storage.googleapis.com/kubernetes-release/release/v1.15.11/bin/linux/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.15.11/bin/linux/amd64/kubectl){: external}
        - **Windows**: Install the Kubernetes CLI in the same directory as the {{site.data.keyword.cloud_notm}} CLI. This setup saves you some file path changes when you run commands later. [https://storage.googleapis.com/kubernetes-release/release/v1.15.11/bin/windows/amd64/kubectl.exe](https://storage.googleapis.com/kubernetes-release/release/v1.15.11/bin/windows/amd64/kubectl.exe){: external}
    * ![Version 4 icon.](images/icon-version-43.png) For clusters that run {{site.data.keyword.redhat_openshift_notm}} 4: Download the `kubectl` version that matches the Kubernetes version of your {{site.data.keyword.redhat_openshift_notm}} cluster. For supported versions, see [{{site.data.keyword.redhat_openshift_notm}} versions](/docs/openshift?topic=openshift-openshift_versions#version_types).
        - **OS X**: [https://storage.googleapis.com/kubernetes-release/release/v1.22.7/bin/darwin/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.22.7/bin/darwin/amd64/kubectl){: external}
        - **Linux**: [https://storage.googleapis.com/kubernetes-release/release/v1.22.7/bin/linux/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.22.7/bin/linux/amd64/kubectl){: external}
        - **Windows**: Install the Kubernetes CLI in the same directory as the {{site.data.keyword.cloud_notm}} CLI. This setup saves you some file path changes when you run commands later. [https://storage.googleapis.com/kubernetes-release/release/v1.22.7/bin/windows/amd64/kubectl.exe](https://storage.googleapis.com/kubernetes-release/release/v1.22.7/bin/windows/amd64/kubectl.exe){: external}

    If you have multiple clusters that run different versions of Kubernetes, you can download separate `kubectl` binary files. Then, set up an alias in your local command-line interface (CLI) profile to point to the separate binary files that match the version of `kubectl` your cluster needs.
    {: tip}

3. Open the downloaded, compressed files. For example, you might double-click the compressed files in your Downloads folder, or run a command such as `tar -xvf oc.tar.gz`.
4. Complete the following steps to add the binary files to your `PATH` system variable.

* **If you use Mac OS or Linux**

1. Move the `oc` and `kubectl` executable files to the `/usr/local/bin` directory.
    ```sh
    mv /<filepath>/oc /usr/local/bin/oc
    ```
    {: pre}

    ```sh
    mv /<filepath>/kubectl /usr/local/bin/kubectl
    ```
    {: pre}

2. Make sure that `/usr/local/bin` is listed in your `PATH` system variable. The `PATH` variable contains all directories where your operating system can find executable files. The directories that are listed in the `PATH` variable serve different purposes. `/usr/local/bin` is used to store executable files for software that is not part of the operating system and that was manually installed by the system administrator.
    ```sh
    echo $PATH
    ```
    {: pre}

    Example CLI output:
    ```sh
    /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
    ```
    {: screen}
        
* **If you use Windows**

1. Move the `oc` and `kubectl` executable files to the `C:\Program Files\IBM\Cloud\bin` directory.
    ```sh
    move "/<filepath>/oc" "C:\Program Files\IBM\Cloud\bin"
    ```
    {: pre}

    ```sh
    move "/<filepath>/kubectl" "C:\Program Files\IBM\Cloud\bin"
    ```
    {: pre}
    
2. Make sure that `C:\Program Files\IBM\Cloud\bin` is listed in your `PATH` system variable. The `PATH` variable contains all directories where your operating system can find executable files. The directories that are listed in the `PATH` variable serve different purposes. `C:\Program Files\IBM\Cloud\bin` is used to store executable files for the IBM Cloud software that is not part of the operating system and that was manually installed by the system administrator.
    ```sh
    echo $PATH
    ```
    {: pre}
    
    Example CLI output:
    ```sh
    PATH:C:\Windows\System32;C:\Windows;C:\Windows\System32\WindowsPowerShell\v1.0\
    ```
    {: screen}
    
    If `C:\Program Files\IBM\Cloud\bin` is not listed, add it to your `PATH` system variable.
    ```sh
    set PATH="%PATH%;C:\Program Files\IBM\Cloud\bin\"
    ```
    {: pre}           

**Optional**: [Enable autocompletion for `kubectl` commands](https://kubernetes.io/docs/tasks/tools/#enabling-shell-autocompletion){: external}. The steps vary depending on the shell that you use. You can repeat the steps to enable autocompletion for `oc` commands. For example, in bash on Linux, instead of `kubectl completion bash >/etc/bash_completion.d/kubectl`, you can run `oc completion bash >/etc/bash_completion.d/oc_completion`.

Next, start [Creating a {{site.data.keyword.openshiftlong_notm}} cluster](/docs/openshift?topic=openshift-openshift_tutorial).

For more information about the `oc` CLI, see the [{{site.data.keyword.redhat_openshift_notm}} documentation](https://docs.openshift.com/container-platform/4.8/cli_reference/openshift_cli/getting-started-cli.html){: external}.
{: note}





## Updating the CLI
{: #cs_cli_upgrade}

Update the CLIs regularly to use new features.
{: shortdesc}

This task includes the information for updating the following CLIs:
- {{site.data.keyword.cloud_notm}} CLI version 0.8.0 or later
- {{site.data.keyword.openshiftlong_notm}} plug-in
- {{site.data.keyword.redhat_openshift_notm}} CLI
- {{site.data.keyword.registrylong_notm}} plug-in


To update the CLIs:

1. Update the {{site.data.keyword.cloud_notm}} CLI. Download the [latest version](/docs/cli?topic=cli-getting-started){: external} and run the installer.

2. Log in to the {{site.data.keyword.cloud_notm}} CLI. Enter your {{site.data.keyword.cloud_notm}} credentials when prompted.

    ```sh
    ibmcloud login
    ```
    {: pre}

    If you have a federated ID, use `ibmcloud login --sso` to log in to the {{site.data.keyword.cloud_notm}} CLI. Enter your username and use the provided URL in your CLI output to retrieve your one-time passcode. You know you have a federated ID when the login fails without the `--sso` and succeeds with the `--sso` option.
    {: tip}

3. Install the update from the {{site.data.keyword.cloud_notm}} plug-in repository.

    ```sh
    ibmcloud plugin update kubernetes-service 
    ```
    {: pre}

4. Verify the plug-in installation by running the following command and checking the list of the plug-ins that are installed.

    ```sh
    ibmcloud plugin list
    ```
    {: pre}

    The {{site.data.keyword.openshiftlong_notm}} plug-in is displayed in the results as `kubernetes-service`.

5. Initialize the CLI.

    ```sh
    ibmcloud oc init
    ```
    {: pre}

6. [Update the {{site.data.keyword.redhat_openshift_notm}} CLI](#cli_oc).

7. Update the `container-registry` plugin from the {{site.data.keyword.cloud_notm}} plug-in repository.

    ```sh
    ibmcloud plugin update container-registry 
    ```
    {: pre}

8. Verify the plug-in installation by running the following command and checking the list of the plug-ins that are installed.

    ```sh
    ibmcloud plugin list
    ```
    {: pre}

    The registry plug-in is displayed in the results as `container-registry`.



## Installing the `odo` CLI for developers
{: #cli-odo-install}

Looking to develop apps without using `kubectl` system admin commands or YAML configuration files? Check out the [Developing on clusters with the {{site.data.keyword.redhat_openshift_notm}} Do CLI](/docs/containers?topic=containers-odo-tutorial) tutorial.
{: shortdesc}



## Uninstalling the CLI
{: #cs_cli_uninstall}

If you no longer need the CLI, you can uninstall it.
{: shortdesc}

This task includes the information for removing these CLIs:


-   {{site.data.keyword.openshiftlong_notm}} plug-in
-   {{site.data.keyword.registrylong_notm}} plug-in

To uninstall the CLIs:

1. Uninstall the {{site.data.keyword.openshiftlong_notm}} plug-in.

    ```sh
    ibmcloud plugin uninstall kubernetes-service
    ```
    {: pre}

2. Uninstall the {{site.data.keyword.registrylong_notm}} plug-in.

    ```sh
    ibmcloud plugin uninstall container-registry
    ```
    {: pre}

4. Verify the plug-ins were uninstalled by running the following command and checking the list of the plug-ins that are installed.

    ```sh
    ibmcloud plugin list
    ```
    {: pre}

    The `kubernetes-service` and the `container-registry` plug-in are not displayed in the results.

5. [Uninstall the {{site.data.keyword.cloud_notm}} CLI.](/docs/cli?topic=cli-uninstall-ibmcloud-cli)

6. Uninstall the Kubernetes CLI.
    ```sh
    sudo rm /usr/local/bin/oc
    ```
    {: pre}



## Using the {{site.data.keyword.cloud-shell_notm}} in your web browser
{: #cloud-shell}

[{{site.data.keyword.cloud-shell_full}}](https://cloud.ibm.com/shell){: external} allows you to use the {{site.data.keyword.cloud_notm}} CLI and various CLI plug-ins to manage your cluster directly from your web browser.
{: shortdesc}

The {{site.data.keyword.cloud-shell_notm}} is enabled with several [plug-ins and tools](/docs/cloud-shell?topic=cloud-shell-plugins-tools), including the base {{site.data.keyword.cloud_notm}} CLI (`ibmcloud`), the {{site.data.keyword.containerlong_notm}} plug-in (`ibmcloud oc`), the {{site.data.keyword.registrylong_notm}} plug-in (`ibmcloud cr`), and the {{site.data.keyword.redhat_openshift_notm}} CLI (`oc`).

While you use the {{site.data.keyword.cloud-shell_short}}, keep in mind the following limitations:
* You can open up to five concurrent sessions, which operate independently so you can work with different resources, regions, and accounts at once.
* Any files that you download and edit locally, such as YAML files, are stored temporarily in the {{site.data.keyword.cloud-shell_short}} and don't persist across sessions.
* {{site.data.keyword.cloud-shell_short}} has a usage quota that limits you to 50 hours within a week.

To launch and use the {{site.data.keyword.cloud-shell_notm}}:

1. In the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/){: external} menu bar, click the {{site.data.keyword.cloud-shell_short}} icon ![{{site.data.keyword.cloud-shell_notm}} icon](../icons/terminal-cloud-shell.svg).
2. A session starts and automatically logs you in to the {{site.data.keyword.cloud_notm}} CLI with your current account credentials.
3. Access your cluster by getting the `oc login` token.

    3. In the [{{site.data.keyword.redhat_openshift_notm}} clusters console](https://cloud.ibm.com/kubernetes/clusters?platformType=openshift){: external}, click the cluster that you want to access.
    4. In the **Actions...** drop-down list, select **Connect via CLI**.
    5. Follow the instructions.

If you can't or don't want to open the {{site.data.keyword.redhat_openshift_notm}} console, you can set the cluster context with the `--admin` flag through the CLI with the `ibmcloud oc cluster config -c <cluster_name_or_ID> --admin` command.
{: note}














