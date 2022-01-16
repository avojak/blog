---
title:  "Deploying the Kubernetes Dashboard"
description: "How to deploy and access the Kubernetes dashboard"
author: avojak
image: https://d33wubrfki0l68.cloudfront.net/349824f68836152722dab89465835e604719caea/6e0b7/images/docs/ui-dashboard.png
tags:
  - software
  - kubernetes
  - evergreen
---

I am constantly forgetting exactly how to deploy and access the Kubernetes dashboard, so this will serve as a note-to-self for future Andrew!

The source of truth for this guide is the official documentation: [Deploy and Access the Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/). However, this writeup will be a tl;dr version of that document.

Also, it should go without saying, but do **NOT** do this for a production Kubernetes cluster.

## Deploying the Dashboard (One-time Setup)

The first step is to deploy the dashboard:

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
```

## Creating a User (One-time Setup)

Once the dashboard is deployed, you will need to create a user for accessing the dashboard. We will create two YAML files and then use `kubectl` to apply them. (See: [Creating sample user](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md))

1. Create the service account

    Create `dashboard-adminuser.yaml`:

    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin-user
      namespace: kubernetes-dashboard
    ```

    Apply:

    ```bash
    $ kubectl apply -f dashboard-adminuser.yaml
    ```

2. Create a ClusterRoleBinding

    Create `dashboard-adminuser-clusterrolebinding.yaml`:

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kubernetes-dashboard
    ```

    Apply:

    ```bash
    $ kubectl apply -f dashboard-adminuser-clusterrolebinding.yaml
    ```

## Accessing the Dashboard

Now that we have a user created, we need to retrieve the token that will be used on the login page of the dashboard:

{% raw %}
```bash
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```
{% endraw %}

The output should be a really long secret that looks like this:

```bash
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Ii\
wia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib\
2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2Vy\
LXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQ\
ubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYW\
Njb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJze\
XN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQli\
tASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hw\
xznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa\
8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1V\
La6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabIN\
X6gP5-Tuuw2svnV6NYQ
```

To actually access the dashboard from our computer, we need to establish a gateway between localhost and the Kubernetes API server. This is done via
the `kubectl proxy` command. You will need to run this command (or ensure that it's running) whenever you want to access the dashboard. Note that the
dashboard will still only be accessible from the machine where you run this command.

```bash
$ kubectl proxy
```

The dashboard will now be available in your web browser at the following URL: [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/).

Simply paste in the token that we retrieved earlier, and you're in!