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
