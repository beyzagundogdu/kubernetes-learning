# Redis-HA Helm Chart Kurulumu (Docker-Desktop for MAC)

## Docker-Desktop Kubernetes Etkinlestirme

Docker-Desktop > Settings > Kubernetes > Enable Kubernetes

. Etkinlestirildikten sonra `kubectl get nodes` komutu ile kontrol edildi.

```
kubectl get nodes
```

cikti:

```
NAME             STATUS  ROLES          AGE  VERSION
docker-desktop   Ready   control-plane  ...  v1.32.2
```

## Helm Repo Ekleme ve Redis-HA Kurulumu

```
helm repo add dandydev https://dnadydeveloper.github.io/charts
helm repo update
```

Kurulum:

```
helm install redis-ha dandydev/redis-ha
```

Kurulumdan sonra pod kontrolu:

```
kubectl get pods
```

## Anty-Affinity Sorunu

Kurulumdan sonra sadece `redis-ha-server-0` calisiyordu. Diger pod'lar `Pending`  durumunda kaldi.

`kubectl describe pod redis-ha-server-1` komutuyla incelendi ve su hata bulundu:

`FailedScheduling: 0/1 nodes are available: 1 node(s) didin't match pod anty-affinity rules.`

> Yani: tek bir node oldugu icin Redis, pod'lari farkli node'lara yaymak istiyor ama bu mumkun degil. Bu durumda Anty-Affinity kurali devre disi birakilmali.


## Anty-Affinity Devre Disi Birakma

### 1- Varsayilan YAML dosyasini al:
```
helm show values dandydev/redis-ha > redis-ha-values.yaml
```

### 2- `hardAntyAffinity: true` satirini bul ve su hale getir:
```
hardAntyAffinity: false
```

### 3- `affinity: |` bolumunu bos birak (override etkisizlessin):
```
affinity: {}
```

### 4- Redis'i yeniden kur:
```
helm uninstall redis-ha
helm install redis-ha -f redis-ha-values.yaml dandydev/redis-ha
```

Cikti:
```
kubectl get pods

NAME               READY  STATUS  RESTARTS  AGE
redis-ha-server-0  3/3    Ready   0         2m
redis-ha-server-1  3/3    Ready   0         2m
redis-ha-server-2  3/3    Ready   0         2m
```


## Redis CLI Ile Test
Redis pod'una baglan:
```
kubectl exec -it redis-ha-server-0 -c redis --sh
```
Redis CLI:
```
redis-cli -h redis-ha.default.svc.cluster.local
set deneme "merhaba dunya" 
get deneme
```

> Sonuc: Artik veri yazilabilir durumda.

## Docker-Desktop Uzerindeki Tek Node Kubernetes Ortaminda redis-ha helm chart'i Ile Redis HA Kurulumu

Avantaj:

. Tek node ortamda Redis cluster gibi davranir.

. Failover ve Sentinel senaryolari test edilebilir.

Dezavantaj:

. Gercek HA saglanmaz (tek node)

. AntiAffinity kaldirildigi icin fault tolerance yoktur.


