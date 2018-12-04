---
title: "PX-Motion with stork on EKS"
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion, eks
description: How to migrate stateful appliations to EKS
hidden: true
---

Pairing with an EKS cluster requires the following additional steps because you
also need to pass in your AWS credentials which will be used to generate the IAM token.

## Create a Secret with your AWS credentials
On the source cluster, create a secret in kube-system namespace with your aws credentials
file:
```
$ kubectl create secret generic --from-file=$HOME/.aws/credentials -n  kube-system aws-creds
secret/aws-creds created
```

## Pass the Secret to stork
Mount the secret created above in the stork deployment. Run `kubectl edit deployment -n kube-system stork` and make the following updates:

* Add the following under spec.template.spec:

```
volumes:
- name: aws-creds
  secret:
       secretName: aws-creds
```

* Add the following under spec.template.spec.containers:

```
volumeMounts:
- mountPath: /root/.aws/
  name: aws-creds
  readOnly: true
```

Save the changes and wait for all the stork pods to be in running state after applying the
changes:
```
$ kubectl get pods -n kube-system -l name=stork
```

