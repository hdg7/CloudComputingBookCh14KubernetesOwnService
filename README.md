# Chapter 14 Own Service on Kubernetes

This repository contains the own service on Kubernetes tutorial mentioned in **Chapter 14** of the book:

[**Cloud Computing for Artificial Intelligence: Concepts, Methods, and Practical Tools**](https://amzn.eu/d/02lMPIKf)

# Adding a Custom Service to a Multi-Service Kubernetes Cluster with Ingress

This guide explains how to add your own service (e.g., a REST API) to an existing multi-service Kubernetes cluster managed with **Kind** and exposed via **Ingress**.

---

## **Prerequisites**
- **Docker** installed and running.
- **Kind** installed ([Kind documentation](https://kind.sigs.k8s.io/)).
- **kubectl** installed and configured.
- Kubernetes manifests for cluster configuration (`cluster-config.yml`), ingress controller (`deploy.yaml`), existing apps (`main-app.yaml`, `foo-app.yaml`, `bar-app.yaml`), your custom app (`restapi-app.yaml`), and ingress rules (`example-ingress.yaml`).

---

## **Steps to Add Your Own Service**

### **1. Create a Kind Cluster**
Use your custom Kind configuration file to create a cluster:

```bash
sudo kind create cluster --name mycluster --config cluster-config.yml
```

This sets up a Kubernetes cluster named `mycluster` using the specified configuration.

---

### **2. Deploy the Ingress Controller**
Apply the manifest that installs the NGINX Ingress controller:

```bash
sudo kubectl apply -f deploy.yaml
```

This ensures that traffic routing via Ingress is supported.

---

### **3. Deploy Existing Applications**
Apply the manifests for the existing services:

```bash
sudo kubectl apply -f main-app.yaml
sudo kubectl apply -f foo-app.yaml
sudo kubectl apply -f bar-app.yaml
```

Each manifest should define a **Deployment** and a **Service** for the respective application.

---

### **4. Build Your Custom Service Image**
Build a Docker image for your custom service (e.g., REST API):

```bash
sudo docker build . -t restapi
```

This command:
- Uses the `Dockerfile` in your current directory.
- Tags the image as `restapi`.

---

### **5. Load the Image into the Kind Cluster**
Make the image available to all nodes in the Kind cluster:

```bash
sudo kind load docker-image restapi --name mycluster
```

---

### **6. Deploy Your Custom Service**
Apply the manifest for your custom service:

```bash
sudo kubectl apply -f restapi-app.yaml
```

This manifest should define:
- A **Deployment** for the REST API pods.
- A **Service** exposing the REST API internally.

---

### **7. Configure Ingress**
Apply the Ingress resource that routes traffic to all services, including your custom service:

```bash
sudo kubectl apply -f example-ingress.yaml
```

Ensure the Ingress manifest includes a rule for your custom service path (e.g., `/restapi/web`).

---

### **8. Test Access**
Verify that all services are accessible via Ingress:

```bash
curl localhost/restapi/web
curl localhost/
curl localhost/foo
curl localhost/bar
```

You should see responses from your custom service and the existing applications.

---

### **9. Delete the Cluster**
When finished, delete the Kind cluster:

```bash
sudo kind delete cluster --name mycluster
```

---

## **Tips**
- To update your custom service, rebuild the image and reload it into Kind:
  ```bash
  docker build . -t restapi
  kind load docker-image restapi --name mycluster
  kubectl apply -f restapi-app.yaml
  ```
- To check the status of pods:
  ```bash
  kubectl get pods
  ```
- To debug Ingress issues, inspect logs of the Ingress controller:
  ```bash
  kubectl logs -n ingress-nginx <controller-pod-name>
  ```

---
