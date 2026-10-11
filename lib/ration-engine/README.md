# 🐂 Rasyon Ustası — Bilimsel Besi Rasyonu Hesaplama Motoru (`lib/ration-engine`)

Bu modül, hiçbir UI veya framework bağımlılığı olmadan **NRC 2016 (Nutrient Requirements of Beef Cattle, 8th Revised Edition)**, **INRA 2018** ve **TAGEM Türkiye Yem Kompozisyon Tabloları** temelinde çalışan bağımsız besi sığırı rasyonu hesaplama ve en az maliyetli dengeleme motorudur.

---

## 1. Bilimsel Formüller ve İnterpolasyon (`nrc-2016-tables.json`)

1. **Büzülmüş Canlı Ağırlık (SBW) ve Yaşama Payı Net Enerjisi ($NE_m$):**
   $$SBW = 0.96 \times CA$$
   $$NE_m \text{ (Mcal/gün)} = 0.077 \times SBW^{0.75}$$
2. **Soğuk / Açık Sundurma Stresi Düzeltmesi:**
   - Hayvan açıkta veya sundurmada barınıyorsa yaşama payı enerjisi ($NE_m$) **+%12** artırılır.
3. **Kuru Madde Tüketimi (KMT / DMI) ve Verim Payı ($NE_g$, $ME$, $MP$, $HP$):**
   - `150 kg` ile `700 kg` canlı ağırlık ve `0.8`, `1.2`, `1.6 kg/gün` canlı ağırlık artışı (ADG) ızgarası üzerinden doğrusal interpolasyon (`interpolateNrc2016`) uygulanır.

---

## 2. Zorunlu Güvenlik Kısıtları (Hard Constraints)

- **Minimum Kaba Yem Oranı:** Rasyon Kuru Maddesinin (KM) **en az %30'u** kaba yemden gelmek zorundadır (işkembe asidozunu önler).
- **Minimum Rasyon NDF Oranı:** Rasyon KM'sinde **NDF ≥ %25** zorunludur.
- **Maksimum Arpa Sınırı:** Kesif yem kuru maddesi içinde **Arpa ≤ %50** ile sınırlandırılır.
- **Yemlik Üre Kısıtı:** `0-6 ay` genç danalarda üre kullanımı **%0** (yasak); 6 ay üstü hayvanlarda toplam KM'nin **maksimum %1'i** ile sınırlandırılır.
- **Kalsiyum / Fosfor ($Ca:P$) Dengesi:** Erkek besi danalarında idrar taşı (sidik zoru / ürolitiazis) oluşumunu engellemek için $Ca:P$ oranı **1.2 : 1 – 2.5 : 1** aralığında tutulur; eksikse otomatik Mermer Tozu ve Vitamin-Mineral Premiks eklenir.

---

## 3. Doğrulanmış 3 Örnek Senaryo Çıktısı

| Senaryo | Canlı Ağırlık & Yaş | Hedef Artış | Kuru Madde (KM) | Günlük Toplam Yaş Yem | Enerji Yoğunluğu | Ham Protein (% KM) | Kaba / Kesif Oranı |
|---|---|---|---|---|---|---|---|
| **1. Başlangıç Besisi** | 250 kg (`6-12 ay`) | 1,2 kg/gün | **6,4 kg KM** | **14,1 kg/gün** | **2.680 kcal/kg** | **%14,2** | **%55 / %45** |
| **2. Orta Dönem Besi** | 380 kg (`1-2 yaş`) | 1,2 kg/gün | **9,0 kg KM** | **16,2 kg/gün** | **2.510 kcal/kg** | **%13,0** | **%55 / %45** |
| **3. Bitirme Besisi** | 550 kg (`2 yaş+`) | 1,6 kg/gün | **12,3 kg KM** | **20,5 kg/gün** | **2.690 kcal/kg** | **%12,8** | **%40 / %60** |

---

## 4. Birim Testlerini Çalıştırma

```bash
node --test lib/ration-engine/beef-engine.test.mjs
```
