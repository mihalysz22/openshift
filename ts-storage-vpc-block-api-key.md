---

copyright: 
  years: 2014, 2022
lastupdated: "2022-03-03"

keywords: openshift, storage

subcollection: openshift

content-type: troubleshoot

---


{{site.data.keyword.attribute-definition-list}}


# {{site.data.keyword.block_storage_is_short}} PVC creation fails after API key reset
{: #vpc-block-api-key-reset-ts}
{: support}

**Infrastructure provider**:
![VPC infrastructure provider icon.](images/icon-vpc-2.svg) VPC


After you reset your API key, {{site.data.keyword.block_storage_is_short}} PVC creation fails with an IAM permission error.
{: tsSymptoms}


Resetting your API key means the credentials the {{site.data.keyword.block_storage_is_short}} add-on uses to provision volumes are no longer valid. After resetting your API key, you must reset the {{site.data.keyword.block_storage_is_short}} controller to use the latest API key for volume provisioning.
{: tsCauses}


After resetting your API key, you must re-create the {{site.data.keyword.block_storage_is_short}} controller pod. To recreate the controller pod, delete it by running the following command:
{: tsResolve}

```sh
oc delete pod -n kube-system ibm-vpc-block-csi-controller-0
```
{: pre}








