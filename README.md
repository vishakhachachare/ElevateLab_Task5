# 🚀 Kubernetes Deployment with Minikube

  
---

##  Objectives

- Set up a **Minikube cluster** locally  
- Deploy a **containerized app** with Kubernetes  
- Expose it via a **service**  
- Scale the app up and down like a boss   
- Inspect pods and logs like a detective   

---

##  Tools Used

- 🐳 **Docker** (as Minikube driver)  
- ☸️ **Minikube**  
- 🧙 **kubectl**

---

# Install on Linux / EC2

#### 1. Install Docker and start it:

```bash
sudo yum install -y docker        # Amazon Linux (or apt on Debian/Ubuntu)
sudo systemctl enable --now docker
```

#### 2. Install kubectl (Linux):

```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client --short
```

#### 3. Install minikube (Linux):

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --driver=docker --memory=3072mb --cpus=2
```

---


##  Step-by-Step Deploy & verify

###  1. Start Your Engines (Minikube)
```
minikube start --driver=docker
```

 2. Deploy the NGINX App
```
kubectl apply -f deployment.yaml
```

 3. Expose the App
```
kubectl apply -f service.yaml
```

4. Scale the pod
```
kubectl scale deployment my-nginx-deployment --replicas=4
```

5. Peek Into a Pod
```
kubectl describe pod <pod-name>
```

---

## How to access the app (multiple ways)
- A — Local Windows / Docker driver (recommended for local dev)

Get minikube IP:
```
minikube ip   # e.g., 192.168.49.2
```

Open in browser:
```
http://<minikube-ip>:30008
# e.g. http://192.168.49.2:30008
```


- B — NodePort on EC2 (when Minikube uses host network / none driver)

If Minikube is running with --driver=none (Kubernetes components on host), NodePort will bind to the host. Then:

```
http://<EC2-PUBLIC-IP>:30008
```
Important: Open Security Group inbound for port 30080/TCP.


- C — LoadBalancer (minikube tunnel — only for local machine)

Apply service-loadbalancer.yaml.

Run locally:
```
sudo minikube tunnel
```

After tunnel starts, kubectl get svc will show an EXTERNAL-IP. Use that IP in browser.

Note: minikube tunnel is intended for local use and typically not usable to assign a public AWS IP when running in EC2.


- D — kubectl port-forward (quick and safe)

This forwards a port from your local machine to the service/pod:

Local bind (default localhost):
```
kubectl port-forward svc/nginx-service 8080:80
# visit http://localhost:8080 on the same machine
```

Bind to all interfaces (expose to external clients on that machine):
```
kubectl port-forward svc/nginx-service 8080:80 --address 0.0.0.0
# visit http://<HOST-IP>:8080 from other machines (if firewall/SG allows)
```

If 0.0.0.0 is used make sure:

- On Linux/EC2: the host firewall + AWS Security Group allows the port (8888).

- On Windows: the Windows firewall allows the port.

If port is already in use choose another port (e.g., 8888) or free the port (see Troubleshooting).

---

## Troubleshooting — quick checklist

- kubectl get pods -A → Are pods in Running state?

- kubectl describe pod <pod> → look for crashloops, liveness/readiness failures.

- kubectl logs <pod> → app logs.

- kubectl get svc → confirm nodePort or external IP.

- If using Docker driver on EC2: NodePort may not be reachable from public IP due to Minikube VM networking. Use SSH tunnel, hostNetwork, or --driver=none (with its caveats).

- If minikube tunnel fails on EC2: it’s usually not appropriate in a cloud VM. Prefer --driver=none or set up a real Kubernetes cluster (kubeadm/EKS) for public LoadBalancer.

- For port conflicts: use lsof/netstat/taskkill to identify and free ports.
