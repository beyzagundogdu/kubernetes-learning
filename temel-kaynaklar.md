
# POD

### Tanim
Kubernetes'te calistirilabilir en kucuk birimdir. Icinde bir veya birden fazla container barindirabilir. Tum containerler ayni network ve storage alanini paylasir.

### Senaryo
Bir Flask uygulaman var. Bu uygulamayi bir container olarak calistiriyorsun. Bu container, pod icine konularak calistirilir.

### Onemli Noktalar

. Podlarin IP'leri gecicidir.

. Pod kendiliginden yeniden baslamaz, silinirse kaybolur.

# REPLICASET

### Tanim
Replicaset, belirli sayida pod'un belirtilen sayida kopyasini (replica) calistirmakla sorumludur. Pod'lardan biri silinirse, otomatik olarak yeni pod yaratir,

### Senaryo
Bir web uygulamasini her zaman 3 instance ile calistirmak istiyorsun. Replicaset bu 3 pod'u canli tutar.

### Onemli Noktalar

. Replicaset dogrudan kullanilmaz, deployment araccigiyla tanimlanir.
 
. `matchlabels` kritik onemdedir.

# DEPLOYMENT

### Tanim
Deployment, Replicaset ve Pod'larin kontrolunu saglar. Versiyon gecisleri, roll-out/roll-back gibi islemleri yonetir.

### Senaryo
Bir web uygulamasinin versiyonunu `v1`'den `v2`'ye gecirmek istiyorsun. Deployment bunu adim adim ve otomatik olarak yapar.

### Onemli Noktalar

. `kubectl apply -f` : Yeni konfigurasyonu uygula

. `kubectl set image` : Canli versiyon degisimi saglar.

. `kubectl rollout status` : Gecis durumunu izler.

# SERVICE

### Tanim
Service, pod'larin IP'si degisse bile sabit bir IP ve DNS adiyla pod'lara erismeyi saglar.

### Senaryo
Frontend uygulaman backend pod'un IP'sine degil, backend-service ismine baglanir.

### Turleri

. Cluster IP (varsayilan, sadece iceriden  erisilebilir)

. NodePort (Node'larin IP'si ve belirli bir port ile disaridan erisim saglar)

. LoadBalancer (bulutta gercek bir IP atar)

### Onemli Noktalar

. targetPort: Pod'un icindeki port

. port: Service'in disariya sundugu port

# CONFIGMAP

### Tanim
Configmap; ortam degiskeni, config dosyasi gibi verileri uygulamaya disaridan saglamak icin kullanilir.

### Senaryo
Bir Spring Boot uygulamasinin icinde GREETING=Hello gibi bir deger gecmek istiyorsun ama bunu container imajinin icine gommek istemiyorsun. Configmap ile bu disaridan yapilir.
 
### Kritik Noktalar

. Ortam degiskeni olarak verilebilir (envFrom ile)

. Volume olarak mount edilebilir (config dosyasi gibi)

. kubectl create configmap ... ile ya da YAML ile olusturulabilir.

. configMapRef ile deployment icinde referans verilir.

# SECRET

### Tanim
Gizli bilgileri (DB sifreleri, API Token'lari) saklamak icin kullanilir. Base64 kodlamasiyla tanimlanir.

### Senaryo
GitHub veya bir DB icin kullanici adi/sifre bilgisi iceren secret'i pod'a volume olarak mount etmek istersin.

### Kritik Noktalar

. `kubectl create secret generic` ile olusturulabilir.

. --from-literal : dogrudan deger girilir.

. base64 ile encode/decode islemi yapilir.

. secretRef ile container ortamina aktarilir.

. Volume olarak /secrets/... gibi mount edilir ve cat komutuyla icerik okunur.

### Gercek Hayat Notu
GitHub tokenini bir secret olarak tanimlayi, CI/CD pipelinelarinda guvenli bir sekilde kullanirsin.

# CANARY DEPLOYMENT

### Tanim
Yeni versiyonun tum kullanicilara yayilmadan once, sadece kucuk bir kismina gonderilmesini saglayan dagitim stratejisidir.

### Senaryo
Bir web uygulamasinin v3 surumunu test etmek istiyorsun ama yalnizca %10 kullaniiciya gitmesini istiyorsun. Ayni service uzerinden 2 Deployment baglayarak bu yapilabilir.

### Yapi

. myboot-v1 ve myboot-v2 adinda iki deployment vardir.

. Her iki deployment da ayni label'i (app:myboot) tasir.

. Service label'a gore pod'lari dagitir.

. Iki deployment farkli replica sayisina sahiptir.

# Ozet Akis

1- Kod yazilir : Dockerfile ile container image olusturulur.

2- Deployment yazilir : Pod'larin kontolu saglanir.

3- ConfigMap/ Secret tanimlanir: Konfigurasyon ayristirilir.

4- Service olusturulur: Uygulama dis dunyaya acilir.

5- Canary veya Rolling deployment yapilir: Versiyon gecisi kontrollu saglanir. 

