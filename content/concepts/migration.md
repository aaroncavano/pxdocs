---
title: Migration
description: Move stateful applications between clusters using PX-Motion
keywords: portworx, cloud, migration, px-motion
weight: 1
series: concepts
---

## Overview
The **PX-Motion** feature lets you migrate your applications and data between
clusters. By using **PX-Motion**, you can get the benefits of portability for your
data-rich workloads in a manner that works with Kubernetes. In a single command,
**PX-Motion** moves Kubernetes-based application specifications, configuration,
and Portworx volumes from one cluster to another. 

Common use cases for **PX-Motion** include:

* **Augment capacity**: Free capacity on critical clusters by evacuating lower
    priority applications to secondary clusters.
* **Blue-green test**: Validate new versions of Kubernetes and/or Portworx using
    both application and its data. This is the same blue-green approach used by
    cloud-native application teams-- now available for your infrastructure.
* **Dev/test**: Promote workloads from dev to staging clusters in an automated
    manner. Thereby, eliminate the manual steps for data preparation that hurt
    fidelity of tests.
* **Lift/Shift**: Move applications and data from an on-prem cluster to a hosted
    AWS EKS or Google GKE. The reverse is also supported to repatriate, move
    applications on-prem.
* **Maintenance**: Decommission a cluster in order to perform hardware-level
    upgrades.

## Pairing clusters
The first step in setting up PX-Motion is to establish a trust relationship between a pair of clusters.
Afterwards, an application namespace user can migrate applications and data within their namespace.
An administrator can migrate on behalf of any Namespace or for all namespaces. 

As an example, the administrator can setup a source cluster (left in the diagram below) to migrate to two
destination clusters (right). Applications can then migrate to each destination cluster. In the example below,
the Blue namespace migrates to Destination Cluster 1 and the Red to Cluster 2. Of course, multiple namespaces
can migrate to a single destination cluster.

![Cluster Pair](/img/cluster-pair.png)

## Migration volumes and applications

Once the cluster pair has been set up you can start migrating volumes and applications between the clusters. 

On Kubernetes, **PX-Motion** leverages Namespaces so that teams can control how
applications get migrated. The overall workflow then allows for teams to
self-service their data needs, all within an administrator’s overall design.

**PX-Motion** on Kubernetes moves application objects, configuration, and data including:

* **Kubernetes Objects**: like Replication Controllers, StatefulSets, and Deployments
which in turn includes the Pod specifications needed to run applications. 
* **Kubernetes Configuration**: like ConfigMaps and Kubernetes Secrets needed for
applications. (External secret stores such as Hashicorp Vault are not moved.) 
* **Portworx volumes**: an incremental snapshot will be compressed and moved from
the source cluster to the destination cluster

The following will **not** be moved as they are either cluster-specific,
external to the cluster, or imprudent:

* **Networking**: external load balancers and overlay networking
* **Service Accounts**
* **Kube-System Namespace**: anything in kube-system namespace will not be moved.

## Getting started  

Steps to set up a cluster pair and perform migration on Kubernetes can be found [here](/portworx-install-with-kubernetes/px-motion). <br>
For other orchestrators, steps can be found [here](/install-with-other/px-motion).

