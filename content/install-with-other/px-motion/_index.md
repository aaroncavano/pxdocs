---
title: "PX-Motion to migrate volumes"
linkTitle: "PX-Motion to migrate volumes"
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: How to migrate volumes across clusters
series: px-motion
aliases:
  - /cloud-references/migration/migration-pxctl.html
  - /cloud-references/migration/migration-pxctl
---

## Overview
This method can be used to migrate volumes between two Portworx clusters. It will
not migrate any scheduler specific resources.

## Pre-requisites
* **Version**: The source AND destination clusters need the v2.0 or later
release of PX-Enterprise on both clusters. As future releases are made, the two
clusters can have different PX-Enterprise versions (e.g. v2.1 and v2.3). Also requires stork v2.0+ on the source cluster.
* **Secret Store** : Make sure you have configured a [secret store](/key-management) on both your clusters.
This will be used to store the credentials for the objectstore.
* **Network Connectivity**: Ports 9001 and 9010 on the destination cluster should be
reachable by the source cluster.

## Pairing clusters

### Get cluster token from destination cluster
On the destination cluster, run the following command from one of the Portworx nodes to get the cluster token:
   `/opt/pwx/bin/pxctl cluster token show`

### Create the cluster pair
On the source cluster create the clusterpair by running the following command:
```
$ /opt/pwx/bin/pxctl cluster pair create --ip <ip_of_destination_cluster_node> --token <token_from_destination_cluster>
```

### Verify creation of cluster pair
If the above step is successful you should see the destination cluster in the list of pairs:
```
$ /opt/pwx/bin/pxctl cluster pair list

CLUSTER-ID                                       NAME            ENDPOINT                     CREDENTIAL-ID
2937523c-a8f6-4564-a683-e3b53b92a3b7 (default)   disrani-px2     http://192.168.56.106:9001   952e15df-ca3e-49df-8c20-92f862a44a78
```

## Migrating Volumes
Once you have created cluster pairs you can migrate volumes to it

### Start migration
Migration can be done at two granularities. If no ClusterID is specified during migration it'll pick up the default cluster pair.

* Migrate all volumes from the cluster:

```
/opt/pwx/bin/pxctl cloudmigrate start --all [ --cluster_id <cluster_id> ]
```
* Migrate a particular volume from the cluster: 

```
/opt/pwx/bin/pxctl cloudmigrate start --volume_id <volumeId> [ --cluster_id <cluster_id> ]
```

### Monitor migrations
3. The status for a migration can be checked by running the following command:

```
$ /opt/pwx/bin/pxctl cloudmigrate status

CLUSTER UUID: 2937523c-a8f6-4564-a683-e3b53b92a3b7
TASK-ID                                  VOLUME-ID           VOLUME-NAME  STAGE  STATUS      LAST-UPDATE
107655ea-0f66-4ffe-99e2-1ef06434aa40     589129994411792979  testVolume   Done   Complete    Sat, 27 Oct 2018 01:12:40 UTC

```
The stages of migration will progress from Backup→ Restore→Done. If any stage fails the status will be marked as Failed.

If the migration is successful you should be able see the volume(s) with the same name created on the destination cluster.

