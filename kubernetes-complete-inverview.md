# TEMELLERI
## API ve REST API Mantigi

. `API`: Uygulamalarin birbirleriyle konusma arayuzu.

. `REST API`: HTTP protokolu uzerinde calisan kaynak tabanli yapi.

. Kubernetes API Server tum objeleri JSON formatinda sunar.

. Ornek API Cagrisi:
```bash
GET /api/v1/namespaces/default/pods
```

## Declarative vs Imperative
 
. Declarative: "Ne istiyorum?" - desired state tanimlarsin.(apply -f)

. Imperative: "Nasil yapilacak?" - adimlari tek tek verirsin.(create/run)

Ornek:
```bash
# imperative
kubectl create deployment nginx --image=nginx

#declarative
yaml -> kubectl apply -f deployment.yaml
```

Avantaj: `Declarative -> Idempotent, GitOps uyumlu`

## Downtime Nedir?

. Servisin erisilemedigi sure

. Kubernetes Deployment rolling update ile zero-downtime saglar.


# Resource ve apiVersion Tablosu

| Resource    | API Group    | apiVersion    |
|-------------|--------------|---------------|
| Pod, Service | (Core)      | v1            |
| Deployment, ReplicaSet, DaemonSet, StatefulSet | apps  | apps/v1  |
| Job, CronJob   | batch  | batch/v1   |
| Ingress     | networking.k8s.io  |  networking.k8s.io/v1  |
| Role, RoleBinding  | rbac.authorization.k8s.io  |  rbac.authorization.k8s.io/v1 |


> Formul: apiVersion = apiGroup/version (Core API icin sadece v1)

# ReplicaSet vs Deployment

| Ozellik     | ReplicaSet     | Deployment     |
|-------------|----------------|----------------|
| Amac        | Pod sayisini sabit tutar.   | Versiyon yonetimi, update, rollback  |
| Rolling Update | Yok     | Var   |
| Rollback    | Yok      | Var  |

## Replicaset YAML
```yaml
apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: my-replicaset
spec: 
  replicas: 3
  selector: 
    matchLabels: 
      app: nginx
  templates:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.18
```

## Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: my-deployment
spec: 
  replicas: 3
  selector: 
    matchLabels: 
      app: nginx
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.18
```

# Selector ve Label Mantigi

### Neden Kritik?

. `selector.matchLabels` ile `template.metadata.labels` eslesmezse ReplicaSet/Deployment Pod'lari yonetemez.

### matchLabels vs matchExpressions

. matchLabels: basit key-value.

```yaml
  selector:
    matchLabels:
      app: nginx
```

. matchExpressions: Esnek kosullar (In, Notin, Exists).

```yaml
  selector:
    matchExpressions:
      - key: app
        operation: In
        values: [nginx, apache]
      - key: tier
        operator: NotIn
        values: [backend]
```

# kubectl edit vs set image vs apply -f

| Komut            | Kullanim Senaryosu             |
|------------------|--------------------------------|
| `kubectl edit`   | Canli objeyi duzelt, hizli mudahale |
| `kubectl set image ` | Hizli image update (CI/CD, hotfix) |
| `kubectl apply -f`   | Prod standardi, GitOps uyumlu  |



### Komutlar 
```bash
kubectl edit deployment my-deployment
kubectl set image deployment/my-deployment nginx=nginx:1.2
kubectl apply -f deployment.yaml
```

# Rolling Update & Rollback
```bash
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment/my-deployment
kubectl rollout unda deployment/my-deployment   
```

# Uygulamali Ornek (Deployment + Service + NetworkPolicy)

### Deployment 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
          image: nginx:latest
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: nginx-service
spec:
  selector:
    app: nginx
  ports: 
    - protocol: TCP
      port: 80
      targetport: 80
```

### NetworkPolicy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namepace-http
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector: {}
      ports:
        - protocol: TCP
          port: 80
```

## Kritik Kavramlar:

. Immutable Pod Spec: Image degisirse eski podlar silinmeli.

. Control Loop: Kubernetes desired state ile actual state'i surekli karsilastirir.

. Label Selector: Service, NetworkPolicy, ReplicaSet hepsi label bazli calisir.

. Zero Downtime: Deployment + Rolling Update ile saglanir.

 
