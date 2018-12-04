---
title: "Set up a Cluster Admin namespace for PX-Motion"
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: How to designate a namespace as the cluster admin namespace
---

By default you can only migrate namespaces in which the Migration object is created.
This is to prevent any user from migrating namespaces for which they do not have access.
You can also designate one namespace as an admin namespace. This will allow an
admin who has access to that namespace to migrate any namespace from the source
cluster.

This requires passing in an additional parameter to the stork deployment with
the admin namespace.

Run the following command to edit the stork deployment:
```
$ kubectl edit deployment -n kube-system stork
```

If `admin-namespace` is your admin namepsace, in the editor, update the arguments to the stork container to specify the
cluster admin namespace using the `--migration-admin-namespace` parameter:
```
      - command:
        - /stork
        - --driver=pxd
        - --verbose
        - --leader-elect=true
        - --migration-admin-namespace=admin-namespace
```

Save the changes and wait for all the stork pods to be in running state after applying the
changes:
```
$ kubectl get pods -n kube-system -l name=stork
```

