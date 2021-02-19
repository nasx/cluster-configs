# Umbrella Cluster Configs

Collection of manifests to be used by Argo CD and ACM for GitOps.

Note: The Application resources and other components needed to support a GitOps workflow have not yet been added to the repository. Each component needs to be manually applied.

## Sealed Secrets Deployment

The deployment of Bitnami's sealed secrets controller is handled during the initial cluster installation. The role to deploy the predefined RSA key pair along with the controller is documented [here](https://github.com/sa-ne/openshift4-vmware-upi).

The `sealed-secrets/` directory in this repository just includes the actual sealed secrets encrypted using the predefined RSA key pair created during the initial cluster deployment.

## Argo CD Deployment

To deploy Argo CD, first install the operator.

```shell
$ oc apply -k argocd-operator/base/
```

Using the Argo CD operator, we can then deploy Argo CD.

```shell
$ oc apply -k argocd/base/
```

## Node Configuration

Various labels and annotations for nodes can be found under the `node-configs/` directory. To apply the node configurations, run the following:

```shell
$ oc apply -k node-configs/
```

Note that in this example, various labels are added but the `node-role.kubernetes.io/worker` label is _removed_ by setting its value to null.

## Other Examples

For the other manifests in this repository, most can be deployed by running `oc apply|create`. In cases where manifests are dependent on a CRD provided by an operator, install the operator first.

### OpenShift Container Storage (OCS) Deployment

To deploy OCS we would first install the operator manifests as follows:

```shell
$ oc create -f ocs-operator/
```

Once the operator comes up, the StorageCluster resource (the _instance_ of OCS) which is dependent on the OCS operator can be created by running:

```shell
$ oc create -f ocs-storagecluster/
```

### Registry Configuration

On UPI based OpenShift installations, the the initial registry deployment is in an unmanaged state. To bring the registry under a managed state we need to provide a PVC and modify the `imageregistry.operator.openshift.io/Config` resource.

#### Create Registry PVC

To create the PVC, run:

```shell
$ oc create -f openshift-image-registry/registry-shared-storage-pvc.yaml
```

#### Bring Registry Under Operator Management

Next we modify the registry configuration to bring it under management and leverage our new PVC. To do this, run:

```shell
$ oc apply -f openshift-image-registry/registry-config.yaml
```