
# Prune Cronjobs

This guide explains how to utilize the clean up scripts

## Prerequisites

1. **Access to OpenShift Cluster**: Ensure you have `oc` CLI installed and authenticated with your OpenShift cluster.
2. **Namespace**: Identify the namespace where you want to deploy the CronJob.
3. **Permissions**: Ensure you have permissions to create ServiceAccounts, RoleBindings, and Secrets.

---

## Step 1: Generate or Use an Existing Kubeconfig

If you already have a `kubeconfig` file, skip to Step 2. To generate a new one:

1. Create a ServiceAccount:
   ```bash
   # Replace <namespace> and <service-account> with your values
   oc create serviceaccount <service-account> -n <namespace>
   ```

2. Grant the ServiceAccount appropriate permissions:
   ```bash
   # Adjust roles as needed (cluster-admin used for simplicity)
   oc adm policy add-cluster-role-to-user cluster-admin -z <service-account> -n <namespace>
   ```

3. Extract the Kubeconfig:
   ```bash
   oc project <namespace>
   SECRET_NAME=$(oc get sa pruner -o jsonpath='{.secrets[0].name}')
   oc get secret $SECRET_NAME -o jsonpath='{.data.kubeconfig}' | base64 -d > kubeconfig
   ```

---

## Step 2: Create a Kubeconfig Secret

With the `kubeconfig` file ready, create a secret:

```bash
oc create secret generic kubeconfig-secret --from-file=kubeconfig=kubeconfig
```

---

## Step 3: Deploy the CronJob

Apply the following YAML configuration. The CronJob dynamically uses the namespace where it is deployed.

```bash
oc apply -f .
```

---

## Step 4: Verify the CronJob

Check if the CronJob is running correctly:

```bash
oc get cronjob
oc get pods
oc logs
```

The logs should show the deletion process of PipelineRuns older than one day.

---

## Notes

- **Dynamic Namespace**: The CronJob uses the namespace where it is deployed, making it reusable across multiple namespaces.
- **Permission Scope**: Adjust the ServiceAccount role (`cluster-admin`) to a more restrictive role if possible for better security.

Let me know if you encounter any issues!
