# StatefulSet Nedir?
Kubernetes'te durum bilgisi (state) tasiyan uygulamalarin yonetilmesini saglayan bir Controller turudur. Bu tur uygulamalar cogu zaman veri tabanlari (PostgreSQL, Cassandra, MongoDB), dagitik dosya sistemleri veya kimligi onemli olan servislerdir.

## StatfulSet ve Deployment Arasindaki Farklar:

| Ozellik     | Deployment       | StatefulSet          |
|-------------|------------------|----------------------|
| Pod isimleri | Rastgele (web-5bcdb96b9f-abcde)  | Sirali (web-0, web-1, web-2 |
| DNS ile erisim | IP'ler degisebilir | Sabit DNS (orn: web-0.web.default.svc.cluster.local) |
| Depolama    | Ayni PVC'yi kullanabilirler | Her pod icin benzersiz PVC olur |
| Silinince davranis | Tum kaynaklar silinir | PVC'ler korunur |
| Baslatma/siralama | Ayni anda baslatilabilir | Sirali baslatma (web-0, sonra web-1) |
| Guncelleme | Hepsi ayni anda guncellenebilir | Tek tek ve sirali guncellenir |


## Temel Bilesenler

### 1.Headless Service (zorunlu)
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: nginx
spec:
  clusterIP: None  # Headless!
  selector:
    app: nginx
  ports:
     - port: 80
     name: web
```
> Headless servis, bir LoadBalancer ya da ClusterIP olusturmaz. Her bir pod'a DNS uzerinden dogrudan erisim saglar:
> nginx-0.nginx.default.svc.cluster.local


### 2.StatefulSet Tanimi

```yaml
apiversion: apps/v1
kind: StatefuSet
metadata:
  name: web
spec:
  serviceName: "nginx"  # Yukaridaki headless servis adi
  replicas: 3
  selector:
    matchlabels:
      app: nginx
    template:
      metadata:
        labels:
          app: nginx 
      spec: 
        terminationGracePeriodSeconds: 10
        containers: 
          - name: nginx
            image: nginx
            ports: 
              - containerPort: 80
            volumeMounts:
              - name: www
                mountPath: /usr/share/nginx/html
    volumeClaimTeplates:
      - metadata:
          name: www
        spec: 
          accessModes: ["ReadWriteOnce"]    
          resources:
            request:
              storage: 1Gi
```

 Aciklama:

. serviceName: Headless servisin adi

. replicas: Olusturulacak pod sayisi (web-0, web-1, web-2)

. volumeClaimTemplates: Her pod icin ootomatik PVC olusturur:

  . www-web-0, www-web-1, www-web-2

. terminationGracePeriodSeconds: Sirayla kapatma islemi icin bekleme suresi


## StatefulSet Neden Gerekli?

**Senaryo: PostgreSQL Cluster**

Bir veritabani dusun: postgres-1 master, digerleri replica.

. Replica pod'larin baslama sirasi onemlidir.

. Her bir pod'un verisi ozeldir ve korunmalidir. 
 
. IP adresleri degil, DNS isimleri sabit olmalidir.

```dns
postgres-0.postgres.default.svc.cluster.local
postgres-1.postgres.default.svc.cluster.local
```

## Volume Kullanimi: Paylasimli mi, Ayri mi?

Her StatefulSet pod'u ayri PVC kullanir:

```yaml
volumeClaimTemplates:
  - name: data
```

PVC'ler: 

```bash
pvc/web-0
pvc/web-1
```

Silinince bu PVC'ler **silinmez**, cunku verinin korunmasi amaclanir.

Amac:

. web-0 > PVC-0 > verisi sabit

. web-1 > PVC-1 > kendi verisi


## StatefulSet ve DNS Mantigi

Headless servis sayesinde her pod kendi DNS servisine sahiptir.
 
```bash
nginx-0.nginx.default.svc.cluster.local
nginx-1.nginx.default.svc.cluster.local
```

Boylece uygulamalar birbirini **isimle** bulabilir.


## StatefulSet Kullanirken Dikkat Edilmesi Gerekenler

| Durum   | Aciklama  |
|---------|-----------|
| selector varsa | Servis tek IP uzerinden tum pod'lara gidebilir. Her pod'a ozel erisim isteniyorsa dikkat edilmeli. |
| Silinince PVC'ler  | StatefulSet silinse bile PVC'ler kalir. Temizlik icin delete pvc ... gerekir. |
| Saglikli silme | Pod'lari once scale ile 0'a indir, sonra StatefulSet sil. |

## Helm ile StatefulSet Kullanimi (Loki Ornegi) 
 
Bircok Helm chart (ornegin Loki), StatefulSet ve PVC kullanimini otomatik olarak yonetir.

```yaml
loki:
  enabled: true
  persistence:
    enabled: true
    storageClass: nfs-client
    size: 1Gi
```

```bash
helm install loki grafana/loki-stack -f values.yaml
```

> Bu yapi sayesinde:
> . StatefulSet olusturulur.
> . PVC olusturulur.
> . Paylasimli NFS Provisioner kullanilir.
> . Volume'lar otomatik baglanir.

## Shared Disk Paylasimi (Deployment ile) 

1.PVC Tanimi

```yaml
apiVersion|: v1
kind: PersistentVolumeClaim 
metadata:
  name: shared-disk
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

2.Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shared-disk-deployment
spec:
  replicas: 2
  selector: 
    matchLabels: 
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec: 
      volumes:
        - name: shared-disk-storage
          persistentVolumeClaim:
            claimName: shared-disk
      containers:
        - name: postgres
          image: 'quay.io/rhdevelopers/myboot:v1'
          volumeMounts:
            - name: shared-disk-storage
              mounthPath: /data
```

> Tum pod'lar ayni diski paylasir.
> Paylasimli yapi gerektiren uygulamalar icin gecerlidir.
> StatefulSet bu durumda kullanilmaz.


## NFS Provisioner ile Otomatik PV Olusturma

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.1.10 \
  --set nfs.path=/exports/data
```

> Artik storageClass: nfs-client yazan PVC'ler, PV'yi otomatik uretir.


# Sonuc

StatefulSet:

. DNS sabitligi, kalici veri, sirali dagitim gibi ihtiyaclari cozer.

. Dagitik, durumlu uygulamalar icin en iyi cozumdur.

. Deployment giib gozukse de detayda bambaska mantikla calisir.

. Pod kimligi, siralamasi ve verisi korunur. 






