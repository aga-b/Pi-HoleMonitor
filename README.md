# Pi-HoleMonitor — Zabbix 6.2 / 7.0 / 7.4

Pi-hole v6 REST API (/api) için ortak kurulum ve alarm kılavuzu.

## Şablonlar

- `templates/pihole-zabbix-6.2.yaml` — Zabbix 6.2.x (6.2.6 dahil)
- `templates/pihole-zabbix-7.0.yaml` — Zabbix 7.0.x
- `templates/pihole-zabbix-7.4.yaml` — Zabbix 7.4.x

Her sürümde aynı API toplama akışı, 65 item, 1 dakikalık ana örnekleme, sorgu, hata ve altyapı metrikleri; kaynak dosyadaki grafikler ve tetikleyiciler korunmuştur. Zabbix sürümüne uygun dosyayı içe aktarın; farklı sürümlerin dosyalarını aynı Zabbix kurulumunda art arda import etmeyin.

## Kurulum

1. **Configuration → Templates → Import** (7.x arayüzünde **Data collection → Templates → Import**) menüsünde uygun YAML'i seçin.
2. Şablonu ilgili Pi-hole host'una bağlayın.
3. Host makrolarına `{$PIHOLE.API.URL}` = `http(s)://<host>:<web-port>/api` ve `{$PIHOLE.PASSWORD}` = Pi-hole v6 web parolasını girin. Parolayı Secret text olarak saklayın.
4. Zabbix server/proxy'nin Pi-hole HTTP API'ye bağlanabildiğinden emin olun. Docker içindeyse yayınlanmış web/API portunu kullanın.
5. Latest data'da `pihole.get.metrics`, `pihole.collector.ok`, `pihole.queries.total` item'larını doğrulayın. `collector.ok=1` geçerli örnek, `0` API/istatistik hatası demektir.

## Alarmlar

| Alarm | Seviye | Koşul |
|---|---|---|
| Pi-hole: no API metrics received for 3 minutes | DISASTER | No raw API metrics received for 3 minutes. |
| Pi-hole: API metrics collection failed | HIGH | REST API authentication, reachability or query-statistics retrieval failed. |
| Pi-hole: no valid DNS statistics for 3 minutes | DISASTER | No valid query statistics during 3-minute sampling window. |
| Pi-hole: no DNS queries processed for 3 minutes | HIGH | No increase in query total for 3 minutes. An idle resolver may also produce this alert. |
| Pi-hole: DNS query stats missing for 3 minutes | HIGH | Dependent query-total item stopped receiving samples. |
| Pi-hole: SERVFAIL DNS failures increasing | HIGH | DNS error count increased in a five-minute window. |
| Pi-hole: REFUSED DNS failures increasing | WARNING | DNS error count increased in a five-minute window. |
| Pi-hole: NOTIMP DNS failures increasing | WARNING | DNS error count increased in a five-minute window. |
| Pi-hole: UNKNOWN DNS failures increasing | WARNING | DNS error count increased in a five-minute window. |

Ana veri toplama periyodu **1 dakika**; DNS sorgusu durgunluğu üç örnekle kontrol edilir. SERVFAIL, REFUSED, NOTIMP ve UNKNOWN hata sayaçlarının 5 dakikalık pencerede artması alarm üretir. Sayacın Pi-hole tarafından sıfırlanması veya kayar pencere sıfırlanması yanlış değerlendirmeye yol açabileceğinden gerçek ortamlarda eşikleri trafik profiline göre ayarlayın.

## Önemli sınırlama

API çalışsa bile port 53 üzerinden UDP/TCP DNS çözümlemesi başarısız olabilir. Ayrıca sorgusuz 3 dakika istemci trafiği olmadığını da gösterebilir. Kesin DNS hizmet sağlığı için farklı bir makineden **aktif UDP ve TCP 53 DNS sorgusu** ve yanıt doğrulaması ayrıca yapılandırılmalıdır. Bu YAML yalnızca Pi-hole REST API verilerini gözlemler.

## Test senaryoları

1. API URL'sini geçici yanlış ayarla: collector=0 / API failure alarmı bekle.
2. Pi-hole API erişimini kes: no API metrics ve/veya collector alarmını doğrula.
3. Kontrollü, sorgu üretilmeyen test ortamında 3 dakikalık sorgu durgunluğu kontrolünü doğrula.
4. Kontrollü olarak başarısız sorgular üret; ilgili hata sayaçlarının ve tetikleyicilerin yükseldiğini kontrol et.
5. Erişimi geri getir ve otomatik toparlanmayı doğrula.

## Not

Dosyalar kaynak Zabbix 7.4 export'una dayanarak sürüm ayrımına göre düzenlendi. Henüz canlı Zabbix 6.2.6 / 7.0 / 7.4 instance'larında import ve entegrasyon testi gerçekleştirilmedi.

## Ayrıntılı teknik notlar


## Kapsam

Bu paket `APP-Pi-hole by HTTP` adlı, Pi-hole v6 REST API (`/api`) kullanan mevcut şablonun Zabbix 6.2 YAML biçimine uyarlanmış sürümüdür. DNS sorguları, cevap/hata türleri, bloklama, istemci sayıları, sunucu metrikleri ve mevcut grafik/tetikleyiciler korunmuştur. Toplam **65 item**, **29 tetikleyici**, **5 grafik** içerir. Yeni sürümde veri alma periyodu **1 dakika**dır.

> Bu dosya, şema ve YAML düzeyinde incelenmiştir; çalışan Zabbix 6.2.6 sunucusuna gerçek import yapılarak doğrulanmamıştır. Pi-hole v5 API ile uyumlu değildir.

## Kurulum

1. Zabbix arayüzünden **Configuration → Templates → Import** sayfasını açın.
2. `Pi-hole_Zabbix_6.2.6.yaml` dosyasını seçin. Yeni nesneleri oluşturma ve mevcut nesneleri güncelleme seçeneklerini açın; **Delete missing** seçeneğini kapalı tutun.
3. `APP-Pi-hole by HTTP` şablonunu DNS sunucusunun Zabbix host'una bağlayın. Pi-hole bir Docker container içinde olabilir; Zabbix server/proxy'nin container API'sine ağ üzerinden erişmesi gerekir.
4. Host makrolarını tanımlayın:
   - `{$PIHOLE.API.URL}` = `http://<dns-host>:<pi-hole-web-port>/api` veya HTTPS adresi (sonunda `/api` olmalı, `/auth` olmamalı).
   - `{$PIHOLE.PASSWORD}` = Pi-hole web arayüzü parolası. **Secret text** olarak girin.
   - CPU/RAM/SWAP eşikleri varsayılan olarak `90` değerindedir.
5. **Latest data** altında önce `pihole.get.metrics`, sonra `pihole.collector.ok` ve `pihole.queries.total` verisini inceleyin. İlk item'da `{"error":"..."}` görünüyorsa API adresi/parola/bağlantı problemidir.

## DNS durumu ve yeni alarmlar

| Alarm | Seviye | Kontrol |
|---|---|---|
| Pi-hole: no API metrics received for 3 minutes | Disaster | `pihole.get.metrics` 3 dakika hiç değer almıyor (`nodata`) |
| Pi-hole: API metrics collection failed | High | API yanıtında hata veya gerekli sorgu istatistikleri eksik (`collector.ok = 0`) |
| Pi-hole: no valid DNS statistics for 3 minutes | Disaster | Son 3 dakikalık pencerede yalnız başarısız istatistik okumaları |
| Pi-hole: no DNS queries processed for 3 minutes | High | Son 3 dakikalık pencerede en az üç ölçüm var ve `queries.total` değişmiyor |
| Pi-hole: DNS query stats missing for 3 minutes | High | Toplam sorgu item'ından 3 dakika değer alınmıyor |
| Pi-hole: SERVFAIL DNS failures increasing | High | SERVFAIL sayacı 5 dakikalık pencerede yükseliyor |
| Pi-hole: REFUSED DNS failures increasing | Warning | REFUSED sayacı 5 dakikalık pencerede yükseliyor |
| Pi-hole: NOTIMP DNS failures increasing | Warning | NOTIMP sayacı 5 dakikalık pencerede yükseliyor |
| Pi-hole: UNKNOWN DNS failures increasing | Warning | UNKNOWN sayacı 5 dakikalık pencerede yükseliyor |

**Önemli sınırlamalar:** DNS sorgusunun olmaması DNS servisinin kesin olarak durduğunu kanıtlamaz; sistemde aktif istemci yoksa da alarm oluşabilir. Ayrıca REST API'nin erişilebilir olması, gerçek UDP/TCP 53 çözümlemesinin çalıştığını garanti etmez. **Gerçek uçtan uca DNS sağlık testi** için farklı bir host/proxy üzerinden belirlenmiş bir alan adına UDP ve TCP 53 sorgusu gönderip hem cevap kodunu hem yanıt süresini kontrol eden ayrıca bir harici kontrol önerilir. Bu şablon yalnızca Pi-hole API'sine erişir, DNS paketleri göndermez. Bu nedenle tüm olası servis arızalarını tek başına kesinlikle tespit edeceği iddia edilmemelidir.

**Sayaçlar:** Pi-hole istatistikleri kayan zaman penceresine ait olabileceğinden, bunların artması tekil hata sayısını doğrudan vermeyebilir. Yeni uyarılar `max > min` kuralıyla sayaç yükselişini tespit eder. Diğer, daha eski statik eşikli tetikleyiciler de korunmuştur. `Unusual high NODATA/NXDOMAIN rate` ifadeleri, eski şablondaki `rate()` işlevinden Zabbix 6.2 için dakika başına değişim yaklaşımına dönüştürülmüştür; bunların eşiklerinin ortamda ayrıca test edilmesi gerekir.

## Doğrulama senaryoları

1. API bağlantısını kesip `pihole.collector.ok = 0` ve API hata alarmını doğrulayın; ulaşılmaz HTTP isteğinde ana Script item JSON hata cevabı verebilir, bu nedenle `nodata` alarmının ayrıca çıkması şart değildir.
2. Pi-hole'a DNS isteği göndermeyi bilinçli olarak durdurup en az üç adet birer dakikalık okumada toplam sayacın değişmediğini doğrulayın; sorgusuz ortamda false-positive ihtimalini değerlendirin.
3. Test amaçlı geçersiz upstream DNS kullanarak SERVFAIL sayısında artış oluşturun; canlı sistemde plansız test yapmayın.
4. Pi-hole erişimini açıp sorgu akışını geri getirin; alarm recovery'sini gözlemleyin.
5. Toplu kullanımda Zabbix server/proxy script timeout ve Pi-hole API yükünü izleyin (eskiden 5 dakika olan API polling artık 1 dakika).

## Güvenlik ve operasyon

- API erişimini yalnızca Zabbix server/proxy kaynaklarından izinli tutun.
- Pi-hole yönetici parolasını YAML'ye yazmayın; Zabbix host seviyesinde secret macro kullanın.
- Bildirim istiyorsanız Zabbix **Alerts → Actions** kısmında `High` ve `Disaster` seviyeleri için aksiyon kurun. Şablonun import edilmesi tek başına e-posta/Telegram bildirimi oluşturmaz.
- API erişiminde HTTPS ve erişim kısıtlamaları tercih edilir.
- Mevcut şablonda DHCP/DNSSEC/bloklama kapalı alarmları vardır. Bu özellikleri isteyerek kapattıysanız ilgili tetikleyicileri host üzerinde devre dışı bırakın.

## Yapılan dönüşümler

- Export version: `7.4` → `6.2`.
- Ana Script item: `5m` → `1m`.
- Mevcut `Get errors` preprocessing adımı eksik/başarısız JSONPath durumunda sağlam çalışacak JavaScript'e dönüştürüldü.
- İki eski `rate()` istatistik ifadesi Zabbix 6.2 ile daha kolay taşınabilecek `max - min` yaklaşımına dönüştürüldü.
- Yeni `pihole.collector.ok` dependent item ve 9 yeni DNS/API tetikleyicisi eklendi.
- Swap CPU benzeri mevcut alarmda `min(...,5)` (5 saniye) kullanımı `min(...,5m)` olarak düzeltildi.
### Zabbix 6.2 dashboard compatibility

Zabbix 6.2 does not support the newer Zabbix 7.4 dashboard widget types (`item`, `piechart`, `svggraph`, `gauge`, `honeycomb`). The 6.2 export intentionally omits the dashboard block while retaining monitored items, triggers, value maps, and classic graphs. Build a dashboard locally with widgets supported by Zabbix 6.2 if desired. Import is not yet verified on a live Zabbix instance.
