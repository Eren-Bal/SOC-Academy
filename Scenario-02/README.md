# SOC Academy - Senaryo 02: Fileless Malware (LotL) & Süreç Ağacı Analizi

## Olay Özeti
Bu laboratuvar çalışmasında, kurumsal ağdaki bir kullanıcıya gönderilen oltalama (phishing) e-postası üzerinden tetiklenen gelişmiş bir **"Dosyasız Zararlı Yazılım" (Fileless Malware)** vakası incelenmiştir. 

Saldırganın sisteme standart bir `.exe` indirmek yerine işletim sisteminin yasal araçlarını (**Living off the Land - LotL**) silah olarak kullandığı bu senaryoda; EDR logları içerisindeki gürültülü veriler arasından süreç hiyerarşisi (Parent-Child Process) analizi **Splunk SIEM** üzerinde gerçekleştirilmiştir.

---

## Adım 1: Karantina Alarmı ve İlk Tespit
Sistemdeki binlerce olağan işletim sistemi süreci (Chrome, Explorer vb.) arasından, EDR sisteminin müdahale edip karantinaya aldığı zararlı aktiviteyi bulmak için filtreleme yapıldı.

**Kullanılan SPL Sorgusu:**
```text
source="advanced_edr_logs.json" action_taken="Quarantined"
| table _time, user.username, device.hostname, process.process_name, alert.threat_name
```

> **Bulgu:** EDR'ın `powershell.exe` üzerinde çalışan "Trojan:PowerShell/ObfuscatedCommand.B" tehdidini tespit edip engellediği ve bu olayın **DESKTOP-ARTH** isimli bilgisayarda gerçekleştiği doğrulandı.

---

## Adım 2: Kök Neden Analizi (Saldırı Zinciri)
PowerShell'in durduk yere neden çalıştığını ve saldırının kaynağını bulmak için kurban bilgisayardaki (**DESKTOP-ARTH**) ebeveyn-çocuk (Parent-Child) süreç ilişkisi incelendi.

**Kullanılan SPL Sorgusu:**
```text
source="advanced_edr_logs.json" device.hostname="DESKTOP-ARTH"
| table _time, process.parent_process_name, process.process_name, process.command_line, action_taken
| sort _time
```

> **Bulgu:** Geriye dönük zaman çizelgesinde saldırı zinciri netleşti: Kullanıcının açtığı Word belgesinin (`WINWORD.EXE`), arka planda komut istemini (`cmd.exe`) çağırdığı, onun da zararlı `powershell.exe` betiğini tetiklediği tespit edildi.

---

## Adım 3: Tehdit Zenginleştirme (Command Line Analizi)
Çalıştırılan komutun amacını anlamak için EDR loglarındaki `command_line` parametresi detaylıca incelendi.

> **Bulgu:** Saldırganın antivirüsleri atlatmak ve analizi zorlaştırmak için komut satırını `-EncodedCommand JABzAD0ATg...` şeklinde **Base64** ile şifrelediği (Obfuscation) görüldü. Bir SOC operasyonunda bu kod CyberChef gibi araçlarla çözülerek asıl zararlı sunucu IP'si tespit edilebilir.

---

## Adım 4: Olay Müdahalesi (Incident Response Playbook)
Bir L1/L2 SOC Analisti perspektifiyle bu ihlale karşı alınması gereken acil aksiyonlar şunlardır:

1. **Ağ İzolasyonu (Containment):** Zararlı kod EDR tarafından durdurulmuş olsa da, drop edilmiş başka bir dosya ihtimaline karşı ilgili makine EDR platformu üzerinden derhal ağdan izole edilmelidir.
2. **Zararlı Temizliği (Eradication):** Saldırının kök nedeni olan oltalama Word dokümanının hash değeri (SHA256) çıkarılmalı ve Mail Gateway üzerinden tüm kurumun posta kutularından temizlenmelidir.
3. **Proaktif Savunma (Detection Engineering):** Ofis programlarının komut satırı araçlarını çalıştırması olağan dışıdır. SIEM üzerinde *"Parent Process: WINWORD.EXE AND Child Process: cmd.exe OR powershell.exe"* mantığıyla çalışan yeni bir korelasyon kuralı yazılmalıdır.
