# Bölüm 16: İleri Seviye Sorun Giderme (Troubleshooting)

Konteyneriniz sürekli kapanıyor (CrashLoopBackOff) ve `docker logs konteyner-id` yazdığınızda ekranda hiçbir hata görmüyorsanız ne yapacaksınız? Sorun muhtemelen uygulamanızda değil, **işletim sisteminin kendisinde veya Docker Daemon'dadır.**

Bu bölümde Linux Kernel tabanlı ve gelişmiş Docker motoru hata ayıklama tekniklerini inceleyeceğiz.

## 1. OOM (Out Of Memory) Çökmelerini Bulmak

Konteyneriniz aşırı RAM tükettiği için Linux çekirdeği (OOM Killer) onu zorla öldürmüş olabilir. Konteyner bu sırada log yazmaya vakit bile bulamaz. Bunu görmek için Linux Kernel loglarına (dmesg) bakmalısınız:

```bash
sudo dmesg -T | grep -i oom
```

Çıktıda `Out of memory: Killed process 4598 (java) total-vm:1500000kB` görüyorsanız, konteynerinize RAM limiti dar gelmiş demektir.

---

## 2. Docker Daemon (Motoru) Logları

Konteynerin değil, **Docker'ın kendi loglarına** bakmak için `journalctl` kullanırız:

```bash
# Docker motorunun son hatalarını göster
sudo journalctl -u docker.service --no-pager -n 50
```

Örneğin diskiniz `overlay2` sürücüsünde inode (dosya sayısı) limitine ulaştıysa veya ağ köprüsü (`docker0`) çöktüyse bu hatayı sadece burada görebilirsiniz.

---

## 3. Ağ Zafiyetleri (Network Troubleshooting)

İki konteyner birbiriyle konuşamıyorsa, DNS mi bozuk yoksa port mu kapalı anlamak için özel bir "Ağ test konteyneri" (Netshoot) kullanırız. Bu imajın içinde `ping`, `curl`, `telnet`, `nslookup` gibi tüm hacker/sistem araçları bulunur.

```bash
docker run -it --rm --network benim-agim nicolaka/netshoot

# İçeri girdikten sonra diğer konteynerin portunu test edin
telnet veritabani-konteyneri 3306
```

---
*Geliştirme ortamları, CI/CD, güvenlik, hata ayıklama... Docker evreninde bilmeniz gereken her şeyi öğrendiniz. Artık sadece Docker komutlarını ezberleyen biri değil, arka planda neyin nasıl çalıştığını bilen, üretim ortamlarına (Production) yön verebilen gerçek bir Docker Hero'sunuz. Yeni nesil yazılım mimarilerinde başarılar dileriz!*
