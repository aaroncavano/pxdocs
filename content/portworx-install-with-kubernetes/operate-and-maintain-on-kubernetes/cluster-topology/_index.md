---
title: Cluster Topology awareness
weight: 4
keywords: portworx, pxctl, command-line tool, cli, reference, kubernetes, geography, locality, rack, zone, region
description: Learn how Portworx nodes can detect where they are placed in the Kubernetes cluster to influence replicas and performance.
noicon: true
series: k8s-op-maintain
---

There are different ways in which you can provide this information to Portworx nodes based on your scheduler. If you want to set the rack, zone and region information using environment variables, [click here](/concepts/update-geography-info).

### Providing rack info to Portworx using node labels {#providing-rack-info-to-portworx-using-node-labels}

Using kubernetes node labels Portworx nodes can be informed about the rack on which they are running. The specific node label to use is `px/rack=rack1`, where `px/rack` is the key, while `rack1` is the value identifying the rack of which the node is a part of. Please make sure the label is a string not starting with a special character or a number.

**Requirements**

Before updating kubernetes node labels double check the permissions listed on Portworx’s ClusterRole `node-get-put-list-role`:

```text
kubectl describe clusterrole node-get-put-list-role
Name:                node-get-put-list-role
Labels:                <none>
Annotations:        <none>
PolicyRule:
  Resources        Non-Resource URLs        Resource Names        Verbs
  ---------        -----------------        --------------        -----
  nodes                []                        []                [get update list]
  pods                 []                        []                [get list]
```

As seen above the permissions on the node object are \[get update list\]. In order for Portworx nodes to dynamically update its rack information from node labels it needs an additional `watch` permission. Update the ClusterRole using

```text
kubectl edit clusterrole node-get-put-list-role
```

Add the `watch` permission and save the edit window.

**Steps to update kubernetes node labels.**

Run the following command to list the existing nodes and their labels.

```text
kubectl get nodes --show-labels
NAME      STATUS    AGE       VERSION   LABELS
vm-1      Ready     14d       v1.7.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=vm-1,node-role.kubernetes.io/master=
vm-2      Ready     14d       v1.7.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=vm-2
vm-3      Ready     14d       v1.7.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=vm-3
```

To indicate node `vm-2` is placed on `rack1` update the node label in the following way:

```text
kubectl label nodes vm-2 px/rack=rack1
node "vm-2" labeled

kubectl get nodes --show-labels
NAME      STATUS    AGE       VERSION   LABELS
vm-1      Ready     14d       v1.7.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=vm-1,node-role.kubernetes.io/master=
vm-2      Ready     14d       v1.7.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=vm-2,px/rack=rack1
vm-3      Ready     14d       v1.7.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=vm-3
[root@vm-1 ~]#

```

kubectl will show the node vm-2 with the new `px/rack` label.

Double check if the rack information is reflected in the PX cluster.

```text
/opt/pwx/bin/pxctl cluster provision-status
NODE        NODE STATUS        POOL        POOL STATUS .....   ZONE           REGION        RACK
vm-2        Online                0        Online      .....   default        default       rack1
vm-3        Online                0        Online      .....   default        default       default

```

The node vm-2 which was labelled `rack1` is reflected on the PX node while the unlabelled node vm-3 is still using the `default` rack info.

All the subsequent updates to the node labels will be automatically picked up by the PX nodes. A deletion of a `px/rack` label will also be reflected.

#### Specifying rack placement for volumes in the storage class {#specifying-rack-placement-for-volumes-in-the-storage-class}

Once the nodes are updated with rack info you can specify how the volume data can spread across your different racks. Following is an example of a storage class that replicates its volume data across racks `rack1` and `rack2`

```text
##### Portworx storage class
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
    name: px-postgres-sc
provisioner: kubernetes.io/portworx-volume
parameters:
   repl: "2"
   shared: "true"
   racks: "rack1,rack2"
```

Any PVC created using the above storage class will have a replication factor of 2 and will have one copy of its data on `rack1` and the other copy on `rack2`
