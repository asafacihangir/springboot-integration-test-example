# Security, UI, Testing and Deployment

### Bu Çalışmada Öğreneceklerimiz

---

Bu çalışmada, REST API'lerini JWT (JSON Web Tokens) ve Spring Security kullanarak güvenli hale getirmeyi ve kullanıcı rollerine göre yetkilendirme yapmayı öğreneceksiniz. Ayrıca, kullanıcı arayüzü (UI) uygulamasının API'leri nasıl tükettiğini görecek ve API'ler için birim ve entegrasyon testlerini otomatikleştirmeyi keşfedeceksiniz. Son olarak, uygulamanızı konteynerleştirip Kubernetes kümesinde dağıtmayı öğreneceksiniz. Bu süreçler, uygulamanızın güvenli, test edilebilir ve ölçeklenebilir olmasını sağlayacak.

---

## Docker İmajı Oluşturma ve Çalıştırma

Spring Boot uygulamanızı konteynerleştirmek ve Docker kullanarak çalıştırmak için aşağıdaki adımları izleyebilirsiniz:

### 1. Adım: Projenizi Temizleyin ve Derleyin

Öncelikle, projenizi temizleyip derlemeniz gerekir. Bu, uygulamanızın en güncel halini almanızı ve gereksiz dosyaları kaldırmanızı sağlar. Bunun için `gradlew` komut dosyasını kullanabilirsiniz:

```bash
./gradlew clean build

```

Bu komut, projedeki geçici dosyaları temizler ve ardından uygulamanızı derler. Derleme işlemi sırasında, proje bağımlılıkları indirilecek ve uygulamanızın çalıştırılabilir bir JAR dosyası oluşturulacaktır.

### 2. Adım: Docker İmajını Oluşturma

Derleme işlemi tamamlandıktan sonra, Spring Boot uygulamanızı bir Docker imajına dönüştürebilirsiniz. Bunun için `bootBuildImage` görevi kullanılır:

```bash
./gradlew bootBuildImage

```

Bu komut, `packt-modern-api-development-chapter09:0.0.1-SNAPSHOT` isimli bir Docker imajı oluşturur. Bu imaj, uygulamanızın tüm bağımlılıkları ve çalıştırılabilir kodlarıyla birlikte paketlenmiş halidir. İmajın oluşturulması, uygulamanızın Docker ortamında çalıştırılabilmesini sağlar.

### 3. Adım: Oluşturulan Docker İmajını Kontrol Etme

Docker imajının başarıyla oluşturulup oluşturulmadığını kontrol etmek için `docker ps` komutunu kullanabilirsiniz:

```bash
docker ps

```

Bu komut, sisteminizde çalışan tüm Docker konteynerlerini listeler. Eğer `packt-modern-api-development-chapter09:0.0.1-SNAPSHOT` isimli imajı görüyorsanız, imaj başarılı bir şekilde oluşturulmuş demektir.

### 4. Adım: Docker İmajını Çalıştırma

Docker imajını çalıştırmak için aşağıdaki komutu kullanabilirsiniz:

```bash
docker run -p 8080:8080 packt-modern-api-development-chapter09:0.0.1-SNAPSHOT

```

Bu komut, `packt-modern-api-development-chapter09:0.0.1-SNAPSHOT` isimli Docker imajını çalıştırır ve uygulamanızın 8080 numaralı port üzerinden erişilebilir olmasını sağlar. `-p 8080:8080` kısmı, Docker konteynerinin içindeki 8080 portunu, makinenizdeki 8080 portuna yönlendirir.

### 5. Adım: Uygulamayı Test Etme

Uygulamanızın doğru bir şekilde çalıştığını test etmek için bir terminalde aşağıdaki komutu çalıştırabilirsiniz:

```bash
curl localhost:8080/actuator/health

```

Bu komut, uygulamanızın sağlık durumu bilgisini döndüren bir HTTP isteği gönderir. Eğer uygulama doğru bir şekilde çalışıyorsa, aşağıdaki gibi bir yanıt almalısınız:

```json
{"status":"UP"}

```

Bu yanıt, uygulamanızın başarılı bir şekilde çalıştığını ve sağlıklı olduğunu gösterir.

---

Bu adımlar, Docker kullanarak Spring Boot uygulamanızı konteynerleştirmek ve yerel makinenizde çalıştırmak için temel bir rehber sunmaktadır. Docker, uygulamanızı daha izole ve taşınabilir hale getirir, böylece farklı ortamlar arasında tutarlılığı sağlar.

## Kubernates’e Deploy Edilmesi

### 1. **Minikube Kurulumu ve Başlatılması**

Minikube'yi kurduktan sonra, Minikube'yi başlatın:

```bash
minikube start

```

### 2. **Docker Daemon'u Minikube'ye Yönlendirme**

`bootBuildImage` komutunu Minikube'nin Docker daemon'unda çalıştırabilmek için Docker'ı Minikube'nin Docker daemon'una yönlendirin:

```bash
eval $(minikube docker-env)

```

Bu komut, Docker işlemlerinizin Minikube'nin içindeki Docker daemon üzerinde çalışmasını sağlar.

### 3. **Docker Image Oluşturma**

Spring Boot uygulamanızın Docker imajını oluşturmak için `bootBuildImage` görevini çalıştırın:

```bash
./gradlew bootBuildImage
```

Bu, Docker imajını Minikube'nin Docker ortamında oluşturacaktır.

### 4. **Kubernetes Manifests Oluşturulması**

Bir `Deployment` ve `Service` tanımlayan YAML dosyaları oluşturun. Aşağıda örnek YAML dosyaları verilmiştir:

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: packt-modern-api-development-chapter09
spec:
  replicas: 1
  selector:
    matchLabels:
      app: packt-modern-api-development-chapter09
  template:
    metadata:
      labels:
        app: packt-modern-api-development-chapter09
    spec:
      containers:
      - name: packt-modern-api-development-chapter09
        image: packt-modern-api-development-chapter09:0.0.1-SNAPSHOT
        ports:
        - containerPort: 8080

```

**Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: packt-modern-api-development-chapter09-service
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30007
  selector:
    app: packt-modern-api-development-chapter09

```

### 5. **Uygulamayı Kubernetes'e Deploy Etmek**

Oluşturduğunuz YAML dosyalarını kullanarak Kubernetes'e deploy edin:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

```

### 6. **Uygulamaya Erişme**

Minikube IP adresini ve node portunu kullanarak uygulamaya erişebilirsiniz. Minikube IP'sini almak için:

```bash
minikube ip

```

Daha sonra tarayıcınızda `http://<Minikube_IP>:30007` adresine giderek uygulamanıza erişebilirsiniz.

### Virtualization(Sanallaştırma) ve Containerization(Konteynerleştirme) Arasındaki Fark Nedir?

**Sanallaştırma**, ana işletim sistemi üzerinde sanal makineler (VM'ler) oluşturmak için kullanılır. Ana sistemin donanım kaynakları, sanal makinelerle paylaşılır. Her sanal makine, kendi işletim sistemini ve uygulamalarını çalıştıran ayrı bir ortam sunar.

**Konteynerleştirme** ise konteynerler oluşturarak, bu konteynerlerin ana donanım ve işletim sistemi üzerinde izole edilmiş süreçler olarak çalıştırılmasını sağlar. Konteynerler, sanal makinelere göre daha hafiftir ve genellikle sadece birkaç MB (bazen GB) gerektirirler. Sanal makineler ise genellikle çok daha ağırdır ve birçok GB alan gerektirirler.

Konteynerler, sanal makinelerden daha hızlı çalışır ve daha taşınabilirdir. Bu nedenle, uygulamaların hızlı bir şekilde dağıtılması ve taşınması gerektiğinde konteynerleştirme genellikle daha avantajlıdır.

### Kubernetes Ne İçin Kullanılır?

Kubernetes, bir konteyner orkestrasyon sistemidir ve uygulama konteynerlerini yönetmek için kullanılır. Kubernetes, çalışan konteynerleri izler, kullanılmadıklarında kapatır ve yetim (bağlantısız) kalan konteynerleri yeniden başlatır. Ayrıca, Kubernetes kümesi ölçeklenebilirlik sağlar. Gerektiğinde CPU, bellek ve depolama gibi kaynakları otomatik olarak sağlayabilir. Bu, uygulamalarınızın ihtiyaç duyduğu kaynakları dinamik olarak yönetmenizi ve ölçeklendirmenizi sağlar.

---

### kubectl Nedir?

kubectl, Kubernetes komut satırı arayüzü (CLI) aracıdır ve bir Kubernetes kümesine karşı komutlar çalıştırmak için kullanılır. kubectl ile Kubernetes kaynaklarını yönetebilirsiniz. Bu bölümde, kubectl'nin `apply` ve `create` komutlarını kullandınız. Bu komutlar, Kubernetes üzerinde uygulamalarınızı ve hizmetlerinizi yönetmenizi sağlar.
