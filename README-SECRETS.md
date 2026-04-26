# NexaFlow Security: Sealed Secrets Guide

This repository uses **Bitnami Sealed Secrets** to store sensitive data securely in Git. Plain Kubernetes Secrets are never committed; instead, we use encrypted `SealedSecret` manifests.

## 1. How it Works
1. The **Sealed Secrets Controller** runs in the `sealed-secrets` namespace.
2. It holds a **private key** that never leaves the cluster.
3. You use the `kubeseal` CLI to encrypt secrets using the controller's **public key**.
4. The resulting `SealedSecret` can be safely pushed to GitHub.
5. In the cluster, the controller decrypts the `SealedSecret` and creates a standard `Secret` for your application.

## 2. Installation
Install the `kubeseal` CLI on your local machine:
- **macOS**: `brew install kubeseal`
- **Linux**: `wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.5/kubeseal-0.24.5-linux-amd64.tar.gz && tar -xvzf kubeseal-...`
- **Windows**: `choco install kubeseal`

## 3. Creating a Sealed Secret
Follow these steps to add or update a secret:

### Step A: Create a local temporary secret
```bash
kubectl create secret generic auth-service-secret \
  --from-literal=SECRET_KEY="your-secret-key" \
  --namespace prod \
  --dry-run=client -o yaml > temp-secret.yaml
```

### Step B: Seal the secret
```bash
kubeseal --format=yaml < temp-secret.yaml > sealed-secret.yaml
```

### Step C: Clean up
```bash
rm temp-secret.yaml  # NEVER commit this file
```

## 4. Environment Handling
- **Dev and Prod use different keys**: Ensure your `kubectl` context is set to the correct cluster before running `kubeseal`.
- **Updating**: To update a secret, repeat the steps above and overwrite the `sealed-secret.yaml` file.

## 5. Deployment Order
The `sealed-secrets` controller is configured with **Sync Wave -5**, ensuring it is always ready to decrypt your secrets before the applications (Wave 0) are deployed.
