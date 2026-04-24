# SOC Academy: SIEM & Incident Response Training Labs

Bu repo, bir "SOC L1 Analisti" bakış açısıyla siber saldırıları tespit etme, analiz etme ve müdahale süreçlerini simüle etmek amacıyla oluşturulmuş bir eğitim ve uygulama alanıdır. 

Sıradan veri setlerinden farklı olarak bu projede; saldırı logları kendi geliştirdiğim "Python tabanlı log jeneratörleri" ile, gerçek dünya ağ trafiğine ve "background noise" (arka plan gürültüsü) dinamiklerine sadık kalınarak üretilmektedir.

---

## Proje Kapsamı ve Metodoloji
Her senaryo, **MITRE ATT&CK** çerçevesine uygun bir saldırı vektörünü temel alır ve şu aşamalardan oluşur:
1. **Log Üretimi:** Python ile gerçekçi Windows/Linux veya Ağ loglarının oluşturulması.
2. **Ingestion:** Verilerin **Splunk SIEM** ortamına aktarılması.
3. **Threat Hunting:** SPL sorguları ile gürültü içerisinden saldırı izlerinin (IoC) tespiti.
4. **Analysis:** Saldırı anatomisinin ve sızma sonrası (post-exploitation) hareketlerin raporlanması.
5. **Playbook:** Olay müdahale adımlarının (Containment, Eradication, Recovery) belirlenmesi.

---

## Kullanılan Teknolojiler
* **SIEM:** Splunk Enterprise
* **Programlama:** Python (Custom Log Generation)
* **Veri Formatı:** JSON / Syslog
* **Framework:** MITRE ATT&CK / NIST Incident Response

---

## Hakkımda
Ben Eren Bal. Siber güvenlik ve savunma odaklı bir Bilgisayar Mühendisliği öğrencisiyim. SOC Analizi, Mavi Takım operasyonları üzerine çalışmalarımı sürdürüyorum.

LinkedIn:(https://www.linkedin.com/in/eren-bal-01a887375/)
---
*Bu proje sürekli güncellenmekte ve yeni senaryolar eklenmektedir.*
