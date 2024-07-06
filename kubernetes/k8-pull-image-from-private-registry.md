# 🎏Pulling a Docker Image from a Private Docker Registry Using Containerd

## Introduction

Need to pull Docker images from a private registry using Containerd? This detailed guide will walk you through the process, ensuring you can securely and efficiently access your private container images.

By the end of this guide, you'll be able to pull images from a private registry using Containerd with confidence and ease. Let's get started!

## Steps 🛵:-

**Step 1** — Create kubernetes secret

```
$ kubectl create secret docker-registry registry-credential --docker-server=docker.io --docker-username=<your-username-of-your-private-registry> --docker-password=<your-password-of-your-private-registry> --docker-email=<your-email>
```

**Step 2** — Modify default service account

Add these codes to the default service-account.

```
$ kubectl edit serviceaccounts default -n <NAMESPACE>
```

Need to add:
```
secrets:
- name: default-token-uudge
imagePullSecrets:
- name: registry-credential
```

Full service-account yaml file

```
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2015-08-07T22:02:39Z
  name: default
  namespace: default
  uid: 052fb0f4-3d50-11e5-b066-42010af0d7b6
secrets:
- name: default-token-uudge
imagePullSecrets:
- name: registry-credential
```

**Step 3** — Add Containerd certification file

```
$ mkdir -p /etc/containerd/certs.d/_default
$ vim /etc/containerd/certs.d/_default/hosts.toml
```

Add the following lines:

```
server = "https://<your-registry-server>"

[host."https://<your-registry-server>"]
  capabilities = ["pull", "resolve", push]
  skip_verify = true # this is optional 
```

**Step 3** — Modify containerd configuration file

```
$ vim /etc/containerd/config.toml
```

locate this line:

```
  [plugins."io.containerd.grpc.v1.cri".registry]
```

and add the following line:

```
  [plugins."io.containerd.grpc.v1.cri".registry]
    config_path = "/etc/containerd/certs.d"
```

Lastly, restart the containerd service

```
systemctl restart containerd
```

## Final Note

If you find this repository useful for learning, please give it a star on GitHub. Thank you!

**Authored by:** [ELemenoppee](https://github.com/ELemenoppee)
