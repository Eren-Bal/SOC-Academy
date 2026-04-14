# SOC Academy - Senaryo 1: Windows RDP Brute Force & Post-Exploitation

## Olay Özeti
Bu laboratuvar çalışmasında, dışarıya açık bırakılmış bir Windows sunucusuna yönelik gerçekleştirilen agresif bir Kaba Kuvvet (Brute Force) saldırısı ve saldırganın sisteme sızdıktan sonraki (Post-Exploitation) hareketleri incelenmiştir. 

Gerçek dünya senaryolarına uygun olarak üretilen log dosyasında arka plan gürültüleri (Process Creation, Normal Logoffs vb.) bulunmakta olup, analiz süreci **Splunk SIEM** üzerinde gerçekleştirilmiştir.

---

## Adım 1: Filtreleme ve İlk Tespit
Sistemdeki binlerce satırlık olağan trafiği eleyip, sadece başarısız oturum açma denemelerini bulmak için filtreleme yapıldı.

**Kullanılan SPL Sorgusu:**
```spl
source="brute_force_training.json" event_id=4625
```

**Bulgu:** Log zaman çizelgesinde saniyeler içinde yüzlerce başarısız giriş denemesi tespit edilerek botnet aktivitesi doğrulandı.

---

## Adım 2: Saldırganın Kimliğini ve Hedefini Belirleme
Bu başarısız girişlerin kim tarafından ve hangi hesaba yapıldığını görmek için istatistik tablosu oluşturuldu.

**Kullanılan SPL Sorgusu:**
```spl
source="brute_force_training.json" event_id=4625
| stats count by source_ip, username
| sort - count
```

**Bulgu:** `85.128.228.204` IP adresinden, sistemin en yetkili hesabı olan `sistem_admin` kullanıcısına yönelik **129 adet** ardışık saldırı tespit edildi.

---

## Adım 3: Hasar Tespiti (Duvar Yıkıldı mı?)
Saldırganın bu denemeler sonucunda parolayı bulup bulmadığını doğrulamak için başarılı giriş (Event ID 4624) logları incelendi.

**Kullanılan SPL Sorgusu:**
```spl
source="brute_force_training.json" source_ip="85.128.228.204" event_id=4624
```

**Bulgu:** İlgili zaman damgasında saldırganın parolayı kırarak `sistem_admin` hesabıyla "başarılı giriş yaptığı" ve sunucuya sızdığı kesinleşti.

---

## Adım 4: Sızma Sonrası Analiz (Post-Exploitation)
Saldırganın içeri girdikten sonra sistemde ne yaptığını görmek için `sistem_admin` hesabının başarılı girişten hemen sonraki hareketleri izlendi.

**Kullanılan SPL Sorgusu:**
```spl
source="brute_force_training.json" username="sistem_admin"
| sort _time
```

**Bulgu:**
1. Saldırgan sızar sızmaz işletim sisteminden özel yetkiler talep etmiş ve almıştır (Event ID 4672).
2. Hemen ardından Komut İstemi'ni çalıştırarak sunucuda komut çalıştırma yetkisi elde etmiştir (Event ID 4688).

---

## Adım 5: Olay Müdahalesi (Incident Response Playbook)
Bir L1/L2 SOC Analisti perspektifiyle bu ihlale karşı alınması gereken acil aksiyonlar şunlardır:

1. Ağ İzolasyonu (Containment): Saldırganın IP adresi (`85.128.228.204`) Firewall ve IPS cihazları üzerinden derhal "Block" edilmelidir.
2. Kimlik Tespiti ve Karantina: `sistem_admin` hesabının parolası acilen sıfırlanmalı, aktif oturumları düşürülmeli ve hesaba Multi-Factor Authentication (MFA) zorunluluğu getirilmelidir.
3. Zararlı Temizliği (Eradication): EDR (Endpoint Detection and Response) platformu üzerinden sunucudaki o şüpheli `cmd.exe` process'i zorla sonlandırılmalı ve sunucuda arka kapı taraması başlatılmalıdır.
