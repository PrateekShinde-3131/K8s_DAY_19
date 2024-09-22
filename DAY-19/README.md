# Kubernetes Pod with ConfigMap and Secret-Injected Environment Variables

## Overview

This project demonstrates how to inject both ConfigMap and Secret values into a Kubernetes Pod using environment variables. The example uses `busybox` containers to showcase both methods:
- **ConfigMaps** for injecting non-sensitive data.
- **Secrets** for injecting sensitive data (such as usernames and passwords).

## Prerequisites

- A Kubernetes cluster (local or cloud-based)
- `kubectl` installed and configured to interact with your cluster
- Basic knowledge of Kubernetes objects like ConfigMaps, Secrets, and Pods

## Steps to Run the Example

### 1. Create the ConfigMap

We will create a ConfigMap named `app-cm` that stores two key-value pairs (`firstname` and `lastname`).

**ConfigMap Definition (`app-configmap.yaml`):**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-cm
data:
  firstname: pratik
  lastname: shinde
```

Apply the ConfigMap using:

```bash
kubectl apply -f app-configmap.yaml
```

### 2. Create the Secret

Next, we'll create a Secret to store sensitive data such as a username and password. Kubernetes Secrets are base64 encoded and can be injected into Pods securely.

**Secret Creation:**

You can create the Secret from the command line:

```bash
kubectl create secret generic db-secret \
  --from-literal=username=mydbuser \
  --from-literal=password=mypassword
```

This creates a Secret called `db-secret` with two key-value pairs: `username` and `password`.

### 3. Create the Pod

The Pod will use the ConfigMap to inject non-sensitive data and the Secret to inject sensitive data as environment variables. It runs a `busybox` container that prints a message and sleeps for 1 hour.

**Pod Definition (`myapp-pod.yaml`):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    env:
    - name: FIRSTNAME
      valueFrom:
        configMapKeyRef:
          name: app-cm
          key: firstname
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

Apply the Pod using:

```bash
kubectl apply -f myapp-pod.yaml
```

### 4. Verify the Pod

Once the Pod is running, you can verify that the container has the correct environment variables injected from both the ConfigMap and the Secret.

To check the logs of the Pod, run:

```bash
kubectl logs myapp-pod
```

You should see the following output:

```
The app is running!
```

The environment variables are set within the container, but sensitive information like the `DB_USERNAME` and `DB_PASSWORD` will not be displayed unless explicitly logged. To verify their presence inside the Pod, you can execute:

```bash
kubectl exec myapp-pod -- printenv | grep DB_
```

### 5. Cleanup

Once you're done, you can delete the Pod, ConfigMap, and Secret by running:

```bash
kubectl delete pod myapp-pod
kubectl delete configmap app-cm
kubectl delete secret db-secret
```

## Key Learnings

- **Kubernetes ConfigMaps**: ConfigMaps store non-sensitive data and can be used to inject configurations into Pods via environment variables or volume mounts.
- **Kubernetes Secrets**: Secrets allow you to securely store and manage sensitive data like passwords, API keys, and certificates. These can be injected into Pods as environment variables or mounted as files.
- **Environment Variable Injection**: Both ConfigMaps and Secrets can be injected as environment variables into a container, enabling the separation of configuration from application code.
- **Pod Execution**: The `sleep 3600` command keeps the container running for an hour, which allows you to verify that everything is working as expected.

## Conclusion

This project demonstrates how to use Kubernetes ConfigMaps and Secrets to manage configuration and sensitive data for applications running in containers. By separating configuration and sensitive information from the application code, you can build more flexible, secure, and maintainable Kubernetes deployments.

