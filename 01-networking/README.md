# Lab 01: Kubernetes Networking

## Overview

Lab ini mempelajari konsep networking di Kubernetes melalui Deployment, Service (ClusterIP, NodePort), dan Ingress.

## Arsitektur

```
Internet
    │
    ▼
┌─────────────┐
│   Ingress   │  nginx.latihan.local → nginx-clusterip
│  (Traefik)  │  backend.latihan.local → backend-svc
└─────────────┘
    │         │
    ▼         ▼
┌────────┐ ┌────────┐
│ nginx  │ │backend │  ClusterIP (akses internal)
│  :80   │ │  :80   │
└────────┘ └────────┘
    │         │
    ▼         ▼
┌────────────────┐
│    Pods        │  nginx-demo (3 replika)
│  nginx / httpd │  backend-demo (2 replika)
└────────────────┘

Alternatif: NodePort
┌─────────────┐
│ nginx:30081 │  Akses langsung dari luar cluster
└─────────────┘
```

## Prasyarat

- Kubernetes cluster berjalan (minikube, kind, atau cloud k8s)
- `kubectl` terinstall dan terkonfigurasi
- Ingress controller terinstall (Traefik/Nginx Ingress)

## Step 1: Deploy Nginx (3 Replicas)

```bash
kubectl apply -f nginx-deploy.yaml
```

**Cek status:**
```bash
kubectl get pods -l app=nginx-demo
kubectl get deployment nginx0deploy
```

**Output yang diharapkan:**
```
NAME                        READY   STATUS    RESTARTS   AGE
nginx0deploy-xxxxx-aaaaa   1/1     Running   0          10s
nginx0deploy-xxxxx-bbbbb   1/1     Running   0          10s
nginx0deploy-xxxxx-ccccc   1/1     Running   0          10s
```

---

## Step 2: Deploy Backend (2 Replicas)

```bash
kubectl apply -f backend-deploy.yaml
```

**Cek status:**
```bash
kubectl get pods -l app=backend-demo
kubectl get deployment backend-deploy
```

---

## Step 3: Service ClusterIP (Akses Internal)

ClusterIP hanya bisa diakses dari dalam cluster. Tidak bisa diakses dari luar.

```bash
kubectl apply -f nginx-svc-clusterip.yaml
kubectl apply -f backend-svc.yaml
```

**Cek service:**
```bash
kubectl get svc
```

**Output yang diharapkan:**
```
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-clusterip  ClusterIP   10.x.x.x       <none>        80/TCP    5s
backend-svc      ClusterIP   10.x.x.x       <none>        80/TCP    5s
```

**Test akses internal:**
```bash
kubectl run test --rm -it --image=busybox -- wget -qO- http://nginx-clusterip
```

---

## Step 4: Service NodePort (Akses External)

NodePort mem暴露 port di setiap node, bisa diakses dari luar cluster.

```bash
kubectl apply -f nginx-svc-nodeport.yaml
```

**Cek service:**
```bash
kubectl get svc nginx-nodeport
```

**Output yang diharapkan:**
```
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-nodeport   NodePort   10.x.x.x      <none>        80:30081/TCP   5s
```

**Akses dari browser:**
```
http://<NODE_IP>:30081
```

**Jika pakai minikube:**
```bash
minikube service nginx-nodeport --url
```

---

## Step 5: Ingress (Routing berdasarkan Hostname)

Ingress memungkinkan routing berdasarkan hostname atau path.

```bash
kubectl apply -f nginx-ingress.yaml
kubectl apply -f backend-ingress.yaml
```

**Cek ingress:**
```bash
kubectl get ingress
```

**Output yang diharapkan:**
```
NAME              CLASS     HOSTS                 ADDRESS          PORTS   AGE
nginx-ingress     traefik   nginx.latihan.local    192.168.49.2     80      5s
backend-ingress   traefik   backend.latihan.local  192.168.49.2     80      5s
```

**Tambahkan hosts di `/etc/hosts` (Linux/Mac) atau `C:\Windows\System32\drivers\etc\hosts` (Windows):**
```
192.168.49.2 nginx.latihan.local backend.latihan.local
```

**Akses dari browser:**
```
http://nginx.latihan.local
http://backend.latihan.local
```

---

## Step 6: Verifikasi & Eksplorasi

### Lihat semua resource
```bash
kubectl get all
```

### Lihat detail endpoint
```bash
kubectl get endpoints nginx-clusterip
kubectl get endpoints backend-svc
```

### Lihat log pod
```bash
kubectl logs -l app=nginx-demo --tail=5
kubectl logs -l app=backend-demo --tail=5
```

### Test koneksi antar service
```bash
kubectl run test --rm -it --image=busybox -- wget -qO- http://backend-svc
```

### Hapus test pod
```bash
kubectl delete pod test --ignore-not-found
```

---

## Step 7: Cleanup

```bash
kubectl delete -f nginx-ingress.yaml
kubectl delete -f backend-ingress.yaml
kubectl delete -f nginx-svc-nodeport.yaml
kubectl delete -f nginx-svc-clusterip.yaml
kubectl delete -f backend-svc.yaml
kubectl delete -f backend-deploy.yaml
kubectl delete -f nginx-deploy.yaml
```

**Atau hapus sekaligus:**
```bash
kubectl delete -f .
```

---

## Ringkasan Konsep

| Konsep | Fungsi | Akses |
|--------|--------|-------|
| **Deployment** | Manage pod replica | - |
| **ClusterIP** | Service internal | Hanya dari dalam cluster |
| **NodePort** | Service external via port | Dari luar via `<NodeIP>:<Port>` |
| **Ingress** | Routing berdasarkan hostname | Dari luar via domain name |

## Referensi

- [Kubernetes Networking](https://kubernetes.io/docs/concepts/services-networking/)
- [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
