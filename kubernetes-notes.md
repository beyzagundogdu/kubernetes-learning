# Kubernetes Temel Notlar ve Komutlar

## Genel Bakis:
. Kubernetes (K8s) bir konteyner orkestrasyon platformudur.
. Amac: Docker gibi konteynerleri olcekli, guvenli, otomatik olarak yonetmek.
. Ozellikleri:
. **Self-Healing**: Pod cokse yeniden baslatir.
. **Scaling**: Trafige gore pod sayisini artirir.
. **Rolling Update**: Yeni versiyonu kesintisiz uygular.
. **Load Balancing**: Servisler ile trafigi dengeler.

## Temel Kavramlar:
| Kavram       | Aciklama     |
|--------------|--------------|
| `Pod`        | Kubernetes'te calisan en kucuk birim. Genelde tek bir container barindirir. |
| `Deployment` | Podlarin kopyalarini yonetir. Olceklendirme, guncelleme, rollback saglar. |
| `Service`    | Podlara erisim icin ag katmani saglar. Trafigi dengeler. |
| `Node`       | Kubernetes'in calistigi fiziksel/VM makine. |
| `Cluster`    | Tum kubernetes sisteminin calistigi ortam (node'lardan olusur). |
| `YAML`       | Kubernetes'e "Ne istiyoruz?" sorusunun cevabini verdigimiz yapilandirma dosyasi. |


## Cluster Kurulumu (K8s) 
. Cluster baslat:
```bash
kubeadm init --apiserver-advertise-address=$(hostname -i) --pod-network-cidr=10.5.0.0/16
```
. kubctl'i kullanmak icin:
```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
export KUBECONFIG=$HOME/.kube/config
```
. Cluster durumunu ogrenmek icin:
```bash
kubectl get nodes
```

## Ag Eklentisi (CNI Plugin)
Podlar birbirine erissin diye. Podlar cluster icinde IP ile konusabilmeli.CNI plugin kurulmazsa, pod’lar IP alamaz, node’lar Ready olmaz, cluster islevsiz kalir.
```bash
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```
## Temel Komutlar
### Pod Olusturma
```bash
kubectl run nginx --image=nginx
```
Tek bir imaj olusturur, nginx docker imajini kullanir.
. Kontrol icin:
```bash
kubectl get pods
```

### Deployment Olusturma
```bash
kubectl cerate deployment my-nginx --image=nginx
```
Deployment nesnesi olusturur.
> Deployment: Podlarin kopya sayisini yonetir, guncellemeleri ve rollbacki detsekler.
Kontrol:
```bash
kubectl get deployments
```

### Podlari ve Deploymentlari Listeleme
```bash
kubectl get pods
kubectl get deployments
```
Detali bilgi:
```bash 
kubectl describe pod <pod-adı>
kubectl describe deployment <deployment-adı>
```

### Pod veya Deployment Silme
```bash
kubectl delete pod <pod-adı>
kubectl delete deployment <deployment-adı>
```

### Servis Olusturma
Deployment icin NodePort tipi servis 
```bash
kubectl expose deployment my-nginx --port=80 --target-port=80 --type=NodePort --name=my-nginx-svc
```
Servisleri listele:
```bash
kubectl get svc
```
## YAML ile Calismak (Infrastucturne as Code)
Ornek Deployment + Service YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21.6
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

Uygula:
```bash
kubectl apply -f nginx-deployment.yaml
```

Kontrol:
```bash
kubectl get pods
kubectl get svc
```

## Guncelleme (Rolling Update)
Yeni versiyona gec:
```bash
vi nginx-deployment.yaml
 # image: nginx:1.21.6 → image: nginx:1.23
kubectl apply -f nginx-deployment.yaml
```
Durumu Izle:
```bash
kubectl rollout status deployment/my-nginx
```

## Rollback
Hatali versiyonu geri al:
```bash
kubectl rollout undo deployment/my-nginx
```
Gecmisi gor:
```bash
kubectl rollout history deployment/my-nginx
```

## Scaling (Manuel Olceklendirme)
Pod sayisini artir:
```bash
kubectl scale deployment my-nginx --replicas=5
```

YAML ile:
```yaml
replicas: 5
```

Sonra:
```bash
kubectl apply -f nginx-deployment.yaml
```
Kontrol:
```bash
kubectl get pods
```

## Autoscaling (HPA)
Otomatik Olceklendirme:
```bash
kubectl autoscale deployment my-nginx --min=2 --max=10 --cpu-percent=50
```

Kontrol:
```bash
kubectl get hpa
```

## Faydali Komutlar
Pod Genis Bilgi:
```bash
kubectl get pods -o wide
```
Kaynak Kullanimi
```bash
kubectl top pods
```

