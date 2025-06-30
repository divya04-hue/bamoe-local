# 🚀 Deploy IBM Business Central on OpenShift Cluster

This guide walks you through deploying **IBM Business Central** (RHPAM) on an OpenShift Cluster using the **Red Hat OpenShift Console** and the **IBM Business Automation Operator**.

---

## 📋 Prerequisites

- Access to OpenShift Console: `https://console-openshift-console.apps.<your-cluster-domain>`
- `kubeadmin` username and password (from Fyre or other source)
- Internet access for pulling OperatorHub resources

---

## 🛠️ Step-by-Step Deployment

### 1. 🔐 Login to OpenShift Console

1. Open the OpenShift Console URL in your browser.
2. Log in using:
   - **Username**: `kubeadmin`
   - **Password**: from your Fyre environment

---

### 2. 🧱 Create a New Project (Namespace)

1. Go to **Home > Projects > Create Project**
2. Set the project name to:

   ```bash
   businesscentral
   ```

3. Click **Create**.

---

### 3. 📦 Install the IBM Business Automation Operator

1. Navigate to: **Operators > OperatorHub**
2. Search for:

   ```
   IBM Business Automation
   ```

3. Click the Operator and then click **Install**.
4. Use **default options**, except:
   - Set **Installation Mode** to _"A specific namespace"_
   - Select the namespace: `businesscentral`
5. Click **Install**.

Wait for the status to show **"Succeeded"**.

---

### 4. ✅ Check Operator Health

1. Go to **Operators > Installed Operators**
2. Confirm the IBM Business Automation Operator shows:
   - `Status: Succeeded`
   - `State: Available`

---

### 5. 🧬 Create a KieApp (Business Central Deployment)

1. Click on the installed **IBM Business Automation** Operator.
2. Navigate to the **KieApp** tab.
3. Click **Create KieApp**.
4. Leave **all values as defaults**, then click **Create**.

---

### 6. 🔍 Retrieve Admin Credentials

After the deployment:

1. Go to **Workloads > Secrets** under the `businesscentral` namespace.
2. Look for a secret similar to:

   ```
   rhpam-admin-secret
   ```

3. Click the secret and decode the base64 encoded fields:
   - `username`
   - `password`

Alternatively, go to the YAML view of the KieApp or Kie server to find the credentials.

---

### 7. 🌐 Access Business Central UI

1. Navigate to **Networking > Routes**
2. Filter by namespace: `businesscentral`
3. Find the route named:

   ```
   rhpamcentr
   ```

4. Copy the route URL.
5. Open it in an **incognito/private window**.

---

### 8. 🔑 Login to Business Central

Use the `username` and `password` obtained in step 6.

You should now be inside the **IBM Business Central dashboard**.

---

## ✅ Post-Deployment Validation

- Go to **Workloads > Pods** and check the status of pods:
  - `rhpamcentr`
  - `rhpam-kieserver`
  - `rhpam-postgresql`
- All pods should show `Status: Running`.
- Go back to **KieApp** tab and verify the **status** section for health info.

---

## 📎 Notes

- Business Central is part of Red Hat Process Automation Manager (RHPAM).
- Use incognito to avoid session cache issues during first-time login.
- For production, consider configuring:
  - External databases
  - Persistent storage (ODF, NFS, etc.)
  - LDAP or SSO integration
  - TLS and secure routes

---

## 🧼 Cleanup (Optional)

If you want to delete the setup:

```bash
oc delete project businesscentral
```

---

## 📘 References

- [Red Hat RHPAM Documentation](https://access.redhat.com/documentation/en-us/red_hat_process_automation_manager)
- [IBM Business Automation Docs](https://www.ibm.com/docs/en/adm)
- [OpenShift OperatorHub](https://operatorhub.io)
