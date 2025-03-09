# How to Create a New User in Argo CD

This guide walks you through the process of creating a new user in Argo CD, configuring Role-Based Access Control (RBAC) permissions, and setting a password for secure access.

By following these steps, you will ensure that the new user has the appropriate permissions while maintaining the security and integrity of your Argo CD environment.

## Prerequisites

+ Access to the Master server.

+ Permissions to interact with the argocd-server pod.


## 1. Verify the Argo CD Server Pod Name

To identify the correct pod running the Argo CD server, execute the following command:

```bash
kubectl get pods -n argocd
```

Example Output:

![alt text](images/image.png)

## 2. Access the Argo CD Server Pod

Once you have identified the correct pod name, enter the pod's shell using:

```bash
kubect exec -it argocd-server-56f7986dff-xv5k7 -n argocd /bin/bash
```

## 3. Log In to Argo CD

To authenticate with Argo CD, use the following command:

```bash
argocd login <url>:<port> --username <username> --password <password>
```

Note: Replace <argo-cd-url>, <port>, <username>, and <password> with your actual Argo CD credentials.

Example Output:

![alt text](images/image-1.png)

## 4. List Existing Argo CD Accounts

To view all existing accounts in Argo CD, run:

```bash
argocd account list
```

This command will display all user accounts, including the default admin account.

![alt text](images/image-2.png)

## 5. Add a New User Account

Argo CD manages users via a ConfigMap. Follow these steps to create a new user:

### Retrieve the ConfigMap

```bash
kubectl get configmap argocd-cm -n argocd -o yaml > argocd-cm.yaml
```

### Edit the ConfigMap

Open the argocd-cm.yaml file and add the following entry under the data section:

```bash
data:
  accounts.testuser: login
```

Example:

![alt text](images/image-3.png)

### Apply the Changes

Save the file and apply the changes using:

```bash
kubectl apply -n argocd  -f argocd-cm.yaml
```

## 6. Verify the New Account

After applying the ConfigMap changes, check if the new user account appears in the list:

```bash
argocd account list
```

If successful, the new account will be visible in the output.

![alt text](images/image-4.png)

## 7. Configure RBAC (Role-Based Access Control)

To assign appropriate permissions to the new user, modify the RBAC configuration.

### Retrieve the RBAC ConfigMap

```bash
kubectl get configmap argocd-rbac-cm -n argocd -o yaml > argocd-rbac-cm.yml
```

### Edit the RBAC ConfigMap

Open argocd-rbac-cm.yml and add a role assignment under the data section:

```bash
data:
  policy.csv: |
    p, role:readonly, applications, get, *, allow
    g, testuser, role:readonly
```

Sample output:

![alt text](images/image-5.png)

Note:

+ readonly grants view-only access.

+ Other roles include readwrite, readexecute, or admin, depending on the access level needed

### Apply the RBAC Changes

Save the file and apply the updated configuration:

```bash
kubectl apply -n argocd -f argocd-rbac-cm.yml
```

## 8. Set a Password for the New User

To finalize the setup, create a password for the new account:

```bash
argocd account update-password --account <new-account-name>
```

Example Output:

![alt text](images/image-6.png)

## 9. Confirm User Login

To test the new account, log in using the newly created credentials:

```bash
argocd login <argo-cd-url>:<port> --username testuser --password <new-password>
```

If successful, the user will gain access based on the assigned RBAC permissions.

## Conclusion

You have successfully created a new user in Argo CD, assigned RBAC permissions, and configured authentication settings. This setup ensures controlled access while maintaining security best practices.

For additional customization, you can modify the RBAC policies to fit specific user roles and project requirements.