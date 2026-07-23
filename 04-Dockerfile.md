# Bölüm 4: Dockerfile ve Kendi İmajlarımızı Oluşturmak

Şimdiye kadar hep başkalarının (örneğin Nginx veya Ubuntu'nun resmi geliştiricilerinin) oluşturduğu imajları indirip çalıştırdık. Ancak gerçek dünyada kendi yazdığımız uygulamaları (Python, Node.js, Java vb.) konteynerleştirmemiz gerekecek.

İşte bu noktada devreye **Dockerfile** girer.

## Dockerfile Nedir?

Dockerfile, Docker'a bir imajın nasıl inşa edileceğini (build) adım adım anlatan, uzantısı olmayan düz bir metin dosyasıdır. Bir yemek tarifi gibi düşünebilirsiniz: Önce işletim sistemini al, içine şu programları kur, benim kodumu içine kopyala ve çalıştır.

### Temel ve İleri Seviye Dockerfile Bileşenleri

Bir Dockerfile içinde yazabileceğiniz tüm komutlar (talimatlar) şunlardır ve kitabımızda tamamı yer almaktadır:

- **`FROM`**: Bu satır, Docker image'ının neye dayanacağını belirler. Yani oluşturulan imajın hangi temel görüntüden başlayacağını belirler. Örneğin: `FROM ubuntu:20.04`
- **`LABEL maintainer`**: Bu satır, Dockerfile'ın kim tarafından yönetildiğini belirtir (`MAINTAINER` komutu kullanımdan kaldırılmıştır). Örneğin: `LABEL maintainer="isimsoyisim email@farukguler.com"`
- **`RUN`**: Bu satır, imajda çalıştırılacak komutları belirler. Örneğin: `RUN apt-get update && apt-get install -y nginx`
- **`CMD`**: Bu satır, konteyner başlatıldığında çalıştırılacak varsayılan komutu belirler. Örneğin: `CMD ["nginx", "-g", "daemon off;"]`
- **`EXPOSE`**: Bu satır, konteynerın hangi portları dinlediğini belirtir (Metadata amaçlıdır, portu gerçekten dışarı açmaz, bunun için `docker run -p` gerekir). Örneğin: `EXPOSE 80 443`
- **`ENV`**: Bu satır, konteyner ortam değişkenlerini belirler. Örneğin: `ENV MY_VAR="my_value"`
- **`COPY`**: Lokal bilgisayarınızdaki dosya veya klasörleri Docker imajına kopyalar (Best practice). Örneğin: `COPY ./src /app`
- **`ADD`**: `COPY` ile aynı işi yapar ancak ekstra olarak internetten (URL) dosya indirebilir ve `.tar`, `.gz` gibi arşivleri kopyalarken otomatik klasöre çıkarabilir.
- **`WORKDIR`**: Bu satır, komutların çalışacağı dizini belirler. Örneğin: `WORKDIR /app`
- **`USER`**: Bu satır, çalışacak kullanıcı adını belirler. Örneğin: `USER nginx`
- **`VOLUME`**: Bu satır, konteynerın bağlanabileceği bir hacim oluşturur. Örneğin: `VOLUME /data`
- **`ARG`**: Bu satır, Dockerfile içinde kullanılacak değişkenleri tanımlar. Bu değişkenler, "docker build" sırasında kullanılabilir. Örneğin: `ARG NODE_VERSION=14`
- **`LABEL`**: Bu satır, imaj hakkında açıklama ve meta verileri eklemek için kullanılır. Örneğin: `LABEL description="This is a custom Docker image for Node.js applications"`
- **`ONBUILD`**: Bu satır, başka bir Dockerfile tarafından kullanılacak işlemleri belirler. Örneğin: `ONBUILD COPY . /app`
- **`HEALTHCHECK`**: Bu satır, konteynerın sağlığını kontrol etmek için kullanılır. Örneğin: `HEALTHCHECK --interval=5m --timeout=3s CMD curl --fail http://localhost:80 || exit 1`
- **`SHELL`**: Bu satır, Dockerfile içinde kullanılacak kabuk (shell) tipini belirler. Örneğin: `SHELL ["/bin/bash", "-c"]`

### Ekstra Dockerfile Komutları (Tamlık Açısından)

Kitabımızda tamlık sağlaması açısından yukarıdakilere ek olarak şu iki komut da yer almaktadır:

- **`ENTRYPOINT`**: Konteyner başlatıldığında çalışacak ana komutu belirler. `CMD`'den farklı olarak dışarıdan gelen parametrelerle kolayca ezilemez. Örneğin: `ENTRYPOINT ["/bin/ping"]`
  > **Önemli:** `ENTRYPOINT` ve `CMD` sıklıkla birlikte kullanılır. Ana uygulama `ENTRYPOINT`'e yazılır, ona gidecek varsayılan parametreler `CMD`'ye yazılır. Örneğin:
  > `ENTRYPOINT ["ping"]`
  > `CMD ["-c", "3", "8.8.8.8"]`
- **`STOPSIGNAL`**: Konteynerin durdurulması istendiğinde uygulamanıza gönderilecek sistem kapatma sinyalini belirler (Zorla kapatan `SIGKILL` yerine uygulamanın temiz kapanmasını sağlayan `SIGTERM` veya `SIGQUIT` önerilir). Örneğin: `STOPSIGNAL SIGTERM`

---

## Örnek: Basit Bir Node.js Uygulamasını Konteynerleştirmek

Diyelim ki `app.js` adında basit bir Node.js dosyanız ve `package.json` dosyanız var. Bunu Dockerize etmek için projenizin ana dizininde `Dockerfile` adında bir dosya oluşturursunuz:

```dockerfile
# 1. Base image olarak Node.js'in hafif (alpine) sürümünü kullanıyoruz
FROM node:18-alpine

# 2. Çalışma dizinini /app olarak belirliyoruz
WORKDIR /app

# 3. Önce sadece package.json'ı kopyalıyoruz (Cache avantajı için)
COPY package.json .

# 4. Bağımlılıkları kuruyoruz (Build anında çalışır)
RUN npm install

# 5. Kalan tüm uygulama kodlarını kopyalıyoruz
COPY . .

# 6. Uygulamanın 3000 portunda çalıştığını belirtiyoruz
EXPOSE 3000

# 7. Konteyner ayağa kalktığında çalışacak komut
CMD ["node", "app.js"]
```

### İmajı İnşa Etmek (Build)

Dockerfile'ı yazdıktan sonra, bu tariften bir İmaj üretmemiz gerekir. Bunun için projenizin (ve Dockerfile'ın) bulunduğu dizinde şu komutu çalıştırırsınız:

```bash
docker build -t benim-node-uygulamam .
```

- `-t` (tag): İmaja vereceğimiz isimdir.
- `.` (nokta): Dockerfile'ın bulunduğu dizini (mevcut dizini) ifade eder.

İnşa süreci bittiğinde `docker images` yazarak kendi imajınızı listede görebilirsiniz.

### Kendi İmajımızı Çalıştırmak

Artık kendi imajımızdan bir konteyner çalıştırabiliriz:

```bash
docker run -d -p 3000:3000 --name node-app benim-node-uygulamam
```

---

## .dockerignore Dosyası

Tıpkı Git'teki `.gitignore` gibi, Docker'ın da imajın içine kopyalamasını **istemediğiniz** dosyaları `.dockerignore` dosyasına yazabilirsiniz.
Örneğin, Node.js projelerinde `node_modules` klasörünün bilgisayarınızdan konteynere kopyalanmasını istemezsiniz, çünkü `RUN npm install` komutuyla zaten konteynerin içinde tertemiz bir şekilde kurulacaktır.

**Örnek `.dockerignore` içeriği:**

```text
node_modules
.git
*.md
```

---

## Multi-Stage Build (Çok Aşamalı İnşa)

Uygulamanızı build ederken (örneğin Java, Go veya React projelerinde) derleyici araçlara (compiler) ihtiyaç duyarsınız. Ancak kod derlendikten sonra, ortaya çıkan nihai dosyayı çalıştırmak için bu ağır araçlara ihtiyacınız yoktur.

Multi-stage build, imaj boyutunu inanılmaz derecede küçültmenizi sağlar.

**Örnek (Go Dili İçin):**

```dockerfile
# BİRİNCİ AŞAMA: BUILDER (Derleyici)
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
# CGO bağımlılıklarını kapatarak statik bir dosya oluşturuyoruz (Alpine ile uyum için)
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp main.go

# İKİNCİ AŞAMA: NİHAİ İMAJ (Çalıştırıcı)
# Çok hafif minimal bir Linux imajı kullanıyoruz (Tamamen boş imaj için 'scratch' de tercih edilebilir)
FROM alpine:latest
WORKDIR /app
# Birinci aşamadaki (builder) derlenmiş dosyayı kopyalıyoruz
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

Bu yöntem sayesinde, içinde koca bir Go dili derleyicisi barındıran 800MB'lık bir imaj yerine, sadece derlenmiş programı barındıran 10MB'lık bir imaj elde edersiniz!

---

## BuildKit ile Gelişmiş Önbellekleme (`RUN --mount`)

Docker BuildKit, modern Dockerfile'larda devrim yaratan önbellekleme (cache) özelliği sunar. `npm install` veya `pip install` gibi paket yükleme komutları her build'de aynı paketleri tekrar tekrar internetten indirmek zorunda kalır. BuildKit'in `--mount=type=cache` özelliği bu sorunu ortadan kaldırır:

```dockerfile
# syntax=docker/dockerfile:1
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./

# BuildKit cache mount: npm paketi indirme klasörünü build'ler arasında önbellekte tutar
RUN --mount=type=cache,target=/root/.npm \
    npm ci --prefer-offline

COPY . .
CMD ["node", "app.js"]
```

Bu sayede ilk build'de indirilen paketler bellekte tutulur ve sonraki build'lerde tekrar indirilmez. Build süresi dakikalardan saniyelere düşebilir!

### Gizli Verileri Build Sırasında Güvenle Kullanmak (`--mount=type=secret`)

Eğer build sırasında bir API anahtarı veya özel bir npm token'ına ihtiyaç duyuyorsanız, bunu `ENV` veya `ARG` ile imaja yazmak **güvenlik açığıdır** (imaj katmanlarında kalır). Bunun yerine:

```dockerfile
# Gizli dosyayı sadece bu RUN komutu süresince (/run/secrets/my_token) göster, imaja yazma
RUN --mount=type=secret,id=my_token \
    NPM_TOKEN=$(cat /run/secrets/my_token) npm ci
```

```bash
# Build sırasında token'ı dışarıdan enjekte etme
docker build --secret id=my_token,src=./my_secret_token.txt -t my-app .
```

> [!TIP]
> BuildKit'i aktifleştirmek için: `DOCKER_BUILDKIT=1 docker build ...` veya Docker Desktop ayarlarından "BuildKit" seçeneğini açın. Docker 23.0+ sürümlerinde BuildKit varsayılan olarak açıktır.

---

## Hızlı Başlangıç: `docker init`

Docker CLI 25+ ile gelen `docker init` komutu, projeniz için otomatik olarak **Dockerfile**, **Compose dosyası** ve **.dockerignore** oluşturur:

```bash
# Proje dizininde çalıştırın
docker init
```

Bu komut size interaktif bir sihirbaz sunar: projenizin dilini (Node.js, Python, Go, Java vb.) sorar ve o dile uygun, best-practice Dockerfile'ı saniyeler içinde hazırlar. Sıfırdan yazmak yerine buradan başlayıp özelleştirmek çok daha hızlıdır.

---
*İmajları oluşturmayı da öğrendik. Ancak konteynerler silindiğinde içindeki verilere ne olur? Sıradaki bölümde Veri Yönetimi (Volumes) konusunu işleyeceğiz.*
