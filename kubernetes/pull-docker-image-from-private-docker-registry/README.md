# 🎏 Pulling a Docker Image from a Private Registry Using Containerd

When working with Containerd, pulling images from a private registry requires proper authentication and configuration. If your registry server requires credentials to access images, you'll need to set up Kubernetes secrets and modify Containerd to recognize and authenticate with your registry.

In this guide, I’ll walk you through the exact steps to configure Containerd for pulling images from a private registry securely. By the end of this tutorial, you’ll have a fully working setup to seamlessly access your private container images. Let’s get started!

## Step 1: Create a Kubernetes Secret for Authentication

If your registry requires credentials to pull images, create a secret using the following command:

```sh
kubectl create secret docker-registry registry-credential \
  --docker-server=<your-registry-server> \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email> -n <namespace>
```

This registry-credential secret securely stores your authentication details.

## Step 2: Link the Secret to the Default Service Account

To ensure Kubernetes uses the secret when pulling images, modify the default service account by running:

```sh
kubectl edit serviceaccounts default -n <NAMESPACE>
```

Add the following configuration under secrets and imagePullSecrets:

```sh
secrets:
  - name: default-token-uudge
imagePullSecrets:
  - name: registry-credential
```

Here’s what your full ServiceAccount YAML should look like:

```sh
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
  namespace: default
  creationTimestamp: 2015-08-07T22:02:39Z
  uid: 052fb0f4-3d50-11e5-b066-42010af0d7b6
secrets:
  - name: default-token-uudge
imagePullSecrets:
  - name: registry-credential
```

This ensures Kubernetes uses the correct credentials when pulling private images.

## Step 3: Configure Containerd to Recognize the Private Registry

### 3.1 Create a Certification File for Containerd

Run the following commands to set up the required directory and create a configuration file:

```sh
mkdir -p /etc/containerd/certs.d/_default
vim /etc/containerd/certs.d/_default/hosts.toml
```

Add the following lines to define your registry server:

```sh
server = "https://<your-registry-server>"

[host."https://<your-registry-server>"]
  capabilities = ["pull", "resolve", "push"]
  skip_verify = true  # Optional: Use only if you are working with self-signed certificates
```

Note: This step is required if your registry server enforces authentication before allowing image pulls.

### 3.2 Modify the Containerd Configuration

Open the Containerd configuration file:

```sh
vim /etc/containerd/config.toml
```

Locate the following section:

```sh
  [plugins."io.containerd.grpc.v1.cri".registry]
```

And add this line inside it:

```sh
    config_path = "/etc/containerd/certs.d"
```

This tells Containerd where to find registry configurations.

## Step 4: Restart the Containerd Service

After making the changes, restart Containerd to apply the new settings:

```sh
systemctl restart containerd
```

## 🎯 Final Thoughts

If your registry server requires authentication to pull images, properly setting up Kubernetes secrets and Containerd configuration is essential. By following these steps, you've ensured that your Kubernetes cluster can securely access private container images.