# Removing a User from ArgoCD

As part of managing user access in ArgoCD, I recently needed to remove a user account to ensure proper security and access control. This guide walks through the step-by-step process I followed to completely remove a user from ArgoCD, including ConfigMap updates, RBAC modifications, and secret cleanup.

## Prerequisites

Before proceeding, I made sure that:

+ I had access to the Master Server where ArgoCD is deployed.

+ I had sufficient permissions to modify ArgoCD configurations.

## Step 1: Accessing the ArgoCD Server Pod

### 1.1 Logging into the Master Server

First, I logged into the server where ArgoCD is running.

### 1.2 Identifying the ArgoCD Server Pod

To get the list of pods in the argocd namespace, I ran:

```bash
kubectl get pods -n argocd
```

Expected Output:

![alt text](images/image.png)

### 1.3 Entering the ArgoCD Server Pod

To access the ArgoCD pod, I executed:

```bash
kubectl exec -it argocd-server-56f7986dff-xv5k7 -n argocd /bin/bash
```

I replaced <argocd-server-pod-name> with the actual pod name from the previous command.

## Step 2: Authenticating with ArgoCD

Once inside the ArgoCD pod, I logged in using my credentials:

```bash
argocd login <url>:<port> --username <username> --password <password>
```

Example Output:

![alt text](images/image-1.png)

## Step 3: Listing Existing User Accounts

To see all current ArgoCD user accounts, I ran:

```bash
argocd account list
```
Example Output:

![alt text](images/image-2.png)

## Step 4: Removing the User from ArgoCD ConfigMap

### 4.1 Retrieving the argocd-cm ConfigMap

I saved the configuration to a local file:

```bash
kubectl get configmap argocd-cm -n argocd -o yaml > argocd-cm.yaml
```

### 4.2 Editing the ConfigMap

I opened the argocd-cm.yaml file and removed the entry for the user under the data section:

```bash
data:
  accounts.testuser: login
```

### 4.3 Applying the Updated ConfigMap

```bash
kubectl apply -n argocd  -f argocd-cm.yaml
```

## Step 5: Verifying User Removal

If I exited the ArgoCD pod, I re-entered it using:

```bash
kubectl exec -it <argocd-server-pod-name> -n argocd -- /bin/bash
```

I then ran the following command to confirm the user was removed:

```bash
argocd account list
```

Expected Output:

![alt text](images/image-3.png)

## Step 6: Updating RBAC Configuration

### 6.1 Retrieving the argocd-rbac-cm ConfigMap

I saved the RBAC configuration to a local file:

```bash
kubectl get configmap argocd-rbac-cm -n argocd -o yaml > argocd-rbac-cm.yml
```

### 6.2 Editing the RBAC Configuration

I removed the user-specific role entry from the data section:

```bash
data:
  policy.csv: |
    p, role:readonly, applications, get, *, allow
    g, crunchops, role:readonly
```

### 6.3 Applying the Updated RBAC Configuration

```bash
kubectl apply -n argocd -f argocd-rbac-cm.yml
```

## Step 7: Removing User Credentials from Secrets

### 7.1 Retrieving the argocd-secret Resource

I saved the secrets to a local file:

```bash
kubectl get secrets argocd-secret -n argocd -o yaml > argocd-secret.yml
```

### 7.2 Editing the Secret Configuration

I opened argocd-secret.yml and removed the user-specific entry under the data section.

![alt text](images/image-4.png)

### 7.3 Applying the Updated Secret Configuration

```bash
kubectl apply -n argocd -f argocd-secret.yml
```

## Conclusion

By following these steps, I successfully removed the user from ArgoCD, ensuring their login credentials, role-based access, and related configurations were fully deleted. Verifying each step helped confirm the changes were correctly applied, maintaining a secure and well-managed ArgoCD environment.


