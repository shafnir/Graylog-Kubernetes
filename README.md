<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b2/Graylog_logo.svg/2560px-Graylog_logo.svg.png?s=800&v=4" alt="Graylog Logo" width="300"/>
</p>

<h1 align="center">Graylog Open Source SIEM on Kubernetes</h1>

<p align="center">
  A fully working Kubernetes deployment of <strong>Graylog Open Source.</strong>
</p>



## 📌 Overview

This setup deploys a minimal Graylog stack with:

- **Graylog Core**
- **Graylog DataNode**
- **Secrets Management**
- **Persistent Volumes (PVCs)** via `hostpath-provisioner`
- **NodePort Exposure**

## Additional Notes

Please review the allocated resources in the PVCs and the mongodb StatefulSet and ensure it matches your requirements.  
This deployment is made for a small lab environment, it is recommened to increase the replicas and the allocated storage in the PVCs.


## ⚙️ Prerequisites

- Kubernetes cluster (e.g. Minikube or real cluster)  
- `kubectl` CLI configured  
- `hostpath-provisioner` installed from [ArtifactHub](https://artifacthub.io/packages/helm/rimusz/hostpath-provisioner)



## 🚀 Deployment Steps

1. Create a namespace named `graylog`:

    ```bash
    kubectl create ns graylog
    ```

2. Create the `graylog-password-secret` used for internal communication:

    ```bash
    kubectl create secret generic -n graylog graylog-password-secret \
      --from-literal=GRAYLOG_PASSWORD_SECRET=$(< /dev/urandom tr -dc A-Z-a-z-0-9 | head -c${1:-96}; echo)
    ```

3. Generate a SHA256 hash of your desired root password:

    ```bash
    echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
    ```

4. Create the `graylog-root-password-sha256` secret using the hash:

    ```bash
    kubectl create secret generic -n graylog graylog-root-password-sha256 \
      --from-literal=GRAYLOG_PASSWORD_SHA256=[YOUR_PASSWORD_HASH]
    ```

5. Clone this repository and apply the manifest files:

    ```bash
    git clone https://github.com/shafnir/Graylog-Kubernetes.git
    cd Graylog-Kubernetes
    kubectl apply -f .
    ```

6. Verify pods are running:

    ```bash
    kubectl get pods -n graylog
    ```

    Example output:
    ```bash
    NAME                        READY   STATUS    RESTARTS   AGE
    datanode-5dcff9cffb-qf26r   1/1     Running   0          57m
    graylog-74558bdf5b-zcc8h    1/1     Running   0          61m
    mongodb-0                   1/1     Running   0          61m
    ```

7. Retrieve the initial admin password from Graylog logs:

    ```bash
    kubectl logs -n graylog graylog-74558bdf5b-zcc8h
    ```

    You should see something like:

    ```bash
    Initial configuration is accessible at 0.0.0.0:9000, with username 'admin' and password 'bhQRFNUvIe'.
    Try clicking on http://admin:bhQRFNUvIe@0.0.0.0:9000
    ```

8. Access the Graylog UI:

    ```
    http://[your-node-ip]:30900
    ```

9. After completing the initial setup wizard, log in with username `admin` and the password you set in **Step 3**.

---
