# Kubernetes Temel Kavramlar ve Bilesenler

## Kubernetes Mimarisi ve Yapisi

### Cluster > Node > Port > Container

. Cluster: Kubernetes'in butunu, birden fazla node'u kapsayan yapi.

. Node: Uygulamalarin calistigi sanal ya da fiziksel sunucu.

. Pod: Kubernetes'in en kucuk calisma birimi. Container'ler burada yasar.

. Container: Gercek anlamda calisan uygulama.

## Kubernetes Duzenekleri (Duzlemler)

### Control Plane

> Kubernetes'in karar verici beyni

. kube-apiserver: Tum REST isteklerini kabul eder, dogrular, etcd'ye yazar. Tum sistemin kapisidir.

. etcd: Kubernetes'in hafizasidir. Tum nesneler (pod,service,config) burada keu-value olarak tutulur.

. kube-scheduler: Yeni pod'lar hangi node'da calisacak karar verir.

. kube-controller-manager: Kubernetes'in durumu hep istenen hale getirmesini saglar (replica, node kontrolu vs).

### Data Plane

> Asil islerin gerceklestigi katman (podlarin calistigi yer)

. kubelet: Her node'da calisir. API server'dan gelen talimatlarla pod'lari baslatir.

. kube-poxy: Servis trafigini dogru poda yonlendirir.IP/port kurallarini yazar.

. container runtime: Pod icindeki container'lari fiilen calistiran motor. orn: containered, CRI-O

### Yardimci Bilesenler ve Network Kavrami

. CoreDNS: Servis adlarini IP'ye cevirir. Orn: `myapp.default.svc.cluster.local`

. CNI(Cointainer Network Interface): Pod'lara IP adresi atar. Pod'lar arasi agi kurar.

. Calio/Cillium/Weave: CNI eklentileridir.Bazilari network policy destegi de saglar.

> CoreDNS ad cevirir, CNI IP verir, kube-proxy trafigi tasir.


## kubectl: Kubernetes'le Konusma Araci

### Ayarlar ve Baglantilar
```
echo $KUBECONFIG                                            # hangi konfigrasyon dosyasinin kullanilacagini gosterir
kubectl config view                                         # aktif olan: cluster, kullanici, namespace, sertifika bilgileri 
kubectl config set-context --current --namespace=mystuff    # default namespace yerine kendi calisma alanini secebilirsin/ tum kubectl komutlarinki artik mystuff namespace'si icinde calistirir.
```

### Kume Gozetleme Komutlari
```
kubectl get nodes --show-labels                       # node'lari listeler. hangi node'a ne ozellik atandigini gorebilirsin.
kubectl get namespaces                                # calisma alanlarini gosterir
kubectl get pods -A -o wide                           # tum namespaceler'de pod'lari detayli gosterir.
````

### Bir Uygulama Yaratmak ve Yonetmek

### Namespace Olustur
```
kubectl create namespace mystuff
```

### Deployment Olustur
```
kubectl create deployment myapp --image=quay.io/rhdevelopers/quarkus-demo1:v1
```

### Olaylari Gozlemle
```
watch kubectl get events --sort-by=.metadata.creationTimestamp
```

## Nesneleri Listele
```
kubectl get deployments
kubectl get replicasets
kubectl get pods --show-labels
```

## Loglara Erisim & Pod Testi
```
kubectl logs -l app=myapp                         # loglara bak
kubectl exec -it <pod-ismi> curl localhost:8080   # pod icinden test
```

## Servisi Dis Dunyaya Acmak
```
kubectl expose deployment myapp --port=8080 --type=LoadBalancer  # bir service olusturur ve uygulamayi disa acar (bulut ortamdaysan LoadBalancer IP'si gelir, Minikube ise NodePort verir)
watch kubectl get services                                       # servisleri izle
```

## Olcekleme ve Guncelleme
```
kubectl scale deployment myapp --replicas=3
kubectl set image deployment/myapp quarkus-demo=quay.io/rhdevelopers/myboot:v2   # rolling update olur pod'lar yavas yavas yeni image'a gecer
```

## Temizlik
```
kubectl delete namespace mystuff                            # bu namespace altindaki her seyi sil
kubectl config set-context --current --namespace=default    # varsayilan namespace'e doner   
```



