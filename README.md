<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b2/Graylog_logo.svg/2560px-Graylog_logo.svg.png?s=800&v=4" alt="Graylog Logo" width="300"/>
</p>

<h1 align="center">Graylog Open Source SIEM on Kubernetes</h1>

<p align="center">
  A fully working Kubernetes deployment of <strong>Graylog Open Source.</strong>
</p>

---

## 📌 Overview


This setup deploys a minimal Graylog stack with:
- **Graylog core**
- **OpenSearch DataNode**
- **Secrets management**
- **Persistent volumes (PVCs)** via `hostpath-provisioner`
- **NodePort exposure**

## ⚙️ Prerequisites

Kubernetes cluster (or minikube)  
kubectl  
hostpath-provisioner installed from <a href="https://artifacthub.io/packages/helm/rimusz/hostpath-provisioner">ArtifactHub</a>

## Deployment Steps


1. Create a namespace named graylog 
```bash
kubectl create ns graylog
```

2. Create the graylog-password-secret for internal communications
```bash
kubectl create secret generic -n graylog graylog-password-secret --from-literal=GRAYLOG_PASSWORD_SECRET=$(< /dev/urandom tr -dc A-Z-a-z-0-9 | head -c${1:-96};echo;)
```

3. Generate a root password hash for the Graylog UI
```bash
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

4. Create the root-password-sha256 secret with the result of the last command (the hash)
```bash
kubectl create secret generic -n graylog graylog-root-password-sha256 --from-literal=GRAYLOG_PASSWORD_SHA256=['YOUR PASSWORD HASH']
```

5. Clone the YAML files into your local folder and apply them with kubectl
```bash
kubectl apply -f .
```

6. Check the pods status
```bash
kubectl get pods -n graylog
```

```bash
NAME                        READY   STATUS    RESTARTS   AGE
datanode-5dcff9cffb-qf26r   1/1     Running   0          57m
graylog-74558bdf5b-zcc8h    1/1     Running   0          61m
mongodb-0                   1/1     Running   0          61m
```

7. Extract the initial login credentials for the UI from the graylog pod
```bash
kubectl logs -n graylog graylog-74558bdf5b-zcc8h # Copy the graylog pod name from above (in your case it will be different)
```
You Should get something like this:
```bash
Initial configuration is accessible at 0.0.0.0:9000, with username 'admin' and password 'bhQRFNUvIe'.
Try clicking on http://admin:bhQRFNUvIe@0.0.0.0:9000
```

8. Access your Graylog UI with http://[node-ip]:30900

9. After completing the preflight setup, use the password you set in step 3 to log in to your Graylog
