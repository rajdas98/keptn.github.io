---
title: Troubleshooting
description: Find tips and tricks to deal with troubles that may occur when using Keptn. 
weight: 100
icon: help
---

In this section, instructions have been summarized that help to troubleshoot known issues that may occur when using Keptn.

## Generating a Support Archive

The Keptn CLI allows to generate a support archive, which can be used as data source for debugging a Keptn installation.
For generating a support archive, please checkout the CLI command [keptn generate support-archive](../reference/cli/commands/keptn_generate_support-archive).

## Keptn API cannot be reached

In rare cases (but especially after a new Keptn installation), the Keptn API cannot be reached.
This prevents e.g. a successful communication between the Keptn CLI and the Keptn API.
In order to solve this problem, please try to restart the `api-gateway-nginx` pod by executing:

```console
kubectl delete pods -n keptn --selector=run=api-gateway-nginx
```

## Verifying a Keptn installation

Especially for troubleshooting purposes, it is necessary to verify that all parts of the Keptn installation are running as intended (i.e., no crashed pods, all distributors running).

<details><summary>Expand instructions</summary>
<p>

- To verify your Keptn installation, retrieve the pods running in the `keptn` namespace.

```console
kubectl get pods -n keptn
```

```console
NAME                                                              READY   STATUS    RESTARTS   AGE
api-gateway-nginx-5669667d4f-2ppg9                                1/1     Running   0          20s
api-service-5b846f4d5b-trmbp                                      1/1     Running   0          28s
bridge-6dcc7cc967-hfvdv                                           1/1     Running   0          21s
configuration-service-589fbfb7d9-2rrmv                            2/2     Running   2          30s
eventbroker-go-7d9bbd5b88-84lgf                                   1/1     Running   0          31s
gatekeeper-service-58d89b6c79-bxzsv                               2/2     Running   2          31s
helm-service-67c6fff6d-qxhsj                                      2/2     Running   0          23s
helm-service-continuous-deployment-distributor-7c4455d697-gwwgj   1/1     Running   3          30s
jmeter-service-5444cc4968-v559v                                   2/2     Running   2          30s
keptn-nats-cluster-0                                              3/3     Running   0          28s
lighthouse-service-65ff48dc57-6hdvx                               2/2     Running   2          30s
mongodb-59975d9f4c-nn5c2                                          1/1     Running   0          26s
mongodb-datastore-7fdb567996-lgjj8                                2/2     Running   2          33s
remediation-service-56777cb979-957l4                              2/2     Running   2          33s
shipyard-service-57c6996f47-pzs9r                                 2/2     Running   2          34s
openshift-route-service-57b45c4dfc-4x5lm                          1/1     Running   0          32s (OpenShift only)
openshift-route-service-create-project-distributor-7d4454cs44xp   1/1     Running   0          33s (OpenShift only)
```

</p></details>

## Installation on Azure aborts
<details><summary>Expand instructions</summary>
<p>

**Investigation:**

The Keptn installation is aborting with the following error:

```console
Cannot obtain the cluster/pod IP CIDR
```

**Reason:** 

The root cause of this issue is that `kubenet` is not used in your AKS cluster. However, it is needed to retrieve the `podCidr` according to the official docs: https://docs.microsoft.com/en-us/rest/api/aks/managedclusters/createorupdate#containerservicenetworkprofile 

**Solution:** 

Please select the **Kubenet network plugin (basic)** when setting up your AKS cluster, instead of *Azure network plugin (advanced)* and retry the installation. You can find more information here: https://docs.microsoft.com/en-us/azure/aks/configure-kubenet 

</p></details>


## Broken Keptn project

When creating a project failed, this can cause a problematic state that manifests in a situation that the Keptn Bridge does not show any project.

<details><summary>Expand instructions</summary>
<p>

**Situation**: Executing [keptn create project](../reference/cli/commands/keptn_create_project) failed with following error messsage: 

```console
Starting to create project   
ID of Keptn context: 9d1a30cd-e00b-4354-a308-03e50368bc40  
Creating project sockshop failed. Could not commit changes.
```

**Problem**: The Keptn Bridge does not show any project even though other projects were already displayed. 

**Solution**: 

* Try to execute the command [keptn delete project](../reference/cli/commands/keptn_delete_project):

* If the command did not work, manually delete the faulty project in the `configuration-service` pod.

    1. Connect to the pod of `configuration-service`: 
    ```console
    kubectl -n keptn exec -it svc/configuration-service sh`
    ```

    1. In the pod, go to: `/data/config/`

    1. Delete the directory with the name of the faulty project: 
    ```console
    rm -rf projectXYZ 
    ```

</p></details>

## Troubleshooting the Installer Job

In some cases, the installer is not running correctly or crashes.

<details><summary>Expand instructions</summary>
<p>

**Investigation:**

The Keptn installation is aborting with an error. The investigation needs to be conducted using the following commands:

* Show all deployed pods in the default namespace (should show the status of the installer pod): ``kubectl get pods``
* Show status of the installer job: ``kubectl get jobs``
* Get logs of the installer job: ``kubectl logs jobs/installer``
* If the installer has partially finished, [verify your Keptn installation](#verifying-a-keptn-installation)

**Possible solutions:**

* If the installer pod shows an ImagePullBackOff error, verify that your cluster can connect to the Internet to pull images (e.g., from docker.io).
* If the installer pod has started, but crashes, please create a [new bug report](https://github.com/keptn/keptn/issues/new?assignees=&labels=bug&template=bug_report.md&title=) with the output of above commands.


</p></details>


## Error: UPGRADE FAILED: timed out waiting for the condition

This error often appears when executing `keptn send event new-artifact` in case of insufficient CPU and/or memory on the Kubernetes cluster.

<details><summary>Expand instructions</summary>
<p>

**Investigation:**

The Helm upgrade runs into a time-out when deploying a new artifact of your service using

```console
keptn send event new-artifact
```

**Reason:** 

In this case, Helm creates a new Kubernetes Deployment with the new artifact, but Kubernetes fails to start the pod. 
Unfortunately, there is no way to catch this error by Helm (right now). A good way to detect the error is to look at the Kubernetes events captured by the cluster:

```console
kubectl -n sockshop-dev get events  --sort-by='.metadata.creationTimestamp'
```

where `sockshop-dev` is the project and stage that you are trying to deploy to.

*Note*: This error can also occur at a later stage (e.g., when using blue-green deployments).

**Solution:** 

Increase the number of vCPUs and/or memory, or add another Kubernetes worker node.

</p></details>


## Helm upgrade runs into a time-out on EKS

Same as the error above, but this issue occurs sometimes using a _single_ worker node on EKS.

**Solution:** 

Increase the number of worker nodes. For example, you can therefore use the `eksctl` CLI:
https://eksctl.io/usage/managing-nodegroups/


## Verify Kubernetes Context with Keptn Installation

If you are performing critical operations, such as installing new Keptn services or upgrading something, please verify
that you are connected to the correct cluster.

* Execute `keptn status` to get the Keptn endpoint: 

```console
keptn status
```

```console
Starting to authenticate
Successfully authenticated
CLI is authenticated against the Keptn cluster http://xx.xx.xx.xx.xip.io/api
```

## NGNIX troubleshooting

If a CLI command like, e.g., `keptn add resource` fails with the following error message:

```
$ keptn add-resource --project=sockshop --service=carts --stage=production --resource=remediation.yaml

Adding resource remediation.yaml to service carts in stage production in project sockshop
Error: Resource remediation.yaml could not be uploaded: invalid character '<' looking for beginning of value
-----DETAILS-----<html>
<head><title>502 Bad Gateway</title><script type="text/javascript" src="/ruxitagentjs_ICA2SVfqru_10191200423105232.js" data-dtconfig="rid=RID_470209891|rpid=-713832838|domain=api.keptn|reportUrl=/rb_bf35021xvs|app=ea7c4b59f27d43eb|featureHash=ICA2SVfqru|rdnt=1|uxrgce=1|bp=2|cuc=k1g1l44n|srms=1,1,,,|uxrgcm=100,25,300,3;100,25,300,3|dpvc=1|bismepl=2000|lastModification=1587774023960|dtVersion=10191200423105232|tp=500,50,0,1|uxdcw=1500|agentUri=/ruxitagentjs_ICA2SVfqru_10191200423105232.js"></script></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.17.9</center>
</body>
</html>
```

You can resolve this problem by restarting the Nginx ingress with the following command:

```
$ kubectl -n keptn delete pod -l run=api-gateway-nginx

pod "api-gateway-nginx-cc948646d-zwrb4" deleted
```

After some seconds, the Nginx ingress pod should be up and running again. You can verify this by executing:

```
$ kubectl get pods -n keptn -l run=api-gateway-nginx

NAME                                READY   STATUS    RESTARTS   AGE
api-gateway-nginx-cc948646d-h6bdb   1/1     Running   0          13m
```

At this point, the CLI commands should work again:

```
$ keptn add-resource --project=sockshop --service=carts --stage=production --resource=remediation.yaml

Adding resource remediation.yaml to service carts in stage production in project sockshop
Resource has been uploaded.
```

## Keptn on Minikube causes a MongoDB issue after a reboot

When rebooting the machine Minikube is installed on, the MongoDB pod in the `keptn` namespace runs in a `CrashLoopBackoff`. 

<details><summary>Expand instructions</summary>
<p>

**Note:** Minikube is a K8s distribution for development environments. Please go with K3s for a more stable setup.

**Investigation:**

* To verify the problem, investigate the logs of the mongodb pod:

```console
kubectl logs -n keptn mongodb-578b4d8bcd-dhgb8
```

```console
=> sourcing /usr/share/container-scripts/mongodb/pre-init//10-check-env-vars.sh ...
=> sourcing /usr/share/container-scripts/mongodb/pre-init//20-setup-wiredtiger-cache.sh ...
=> sourcing /usr/share/container-scripts/mongodb/pre-init//30-set-config-file.sh ...
=> sourcing /usr/share/container-scripts/mongodb/pre-init//35-setup-default-datadir.sh ...
ERROR: Couldn't write into /var/lib/mongodb/data
CAUSE: current user doesn't have permissions for writing to /var/lib/mongodb/data directory
DETAILS: current user id = 184, user groups: 184 0
stat: failed to get security context of '/var/lib/mongodb/data': No data available
DETAILS: directory permissions: drwxr-xr-x owned by 0:0, SELinux: ?
```

**Reason:** 

The problem is a permission issue on the `/var/lib/mongodb/data` folder. See [kubernetes/minikube#1184](https://github.com/kubernetes/minikube/issues/1184) and Minikube 'none' driver: https://minikube.sigs.k8s.io/docs/reference/drivers/none/ which lay out complexity for persistence.

**Solution:** 

A workaround for this issue is to add an `initContainer` to the mongodb deployment as shown below. This container will be executed before the actual mongodb container and sets the right permissions on the `/var/lib/mongodb/data` folder. 

```yaml
initContainers:
- name: volume-mount-hack
    image: busybox
    command: ["sh", "-c", "chown -R 184:184 /var/lib/mongodb/data"]
    volumeMounts:
    - name: mongodata
      mountPath: /var/lib/mongodb/data
```

</p></details>