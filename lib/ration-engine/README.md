# Bilimsel Rasyon Hesaplama Motoru (`lib/ration-engine`)

Bu modül, **Süt Ustası (Sağmal & Kuru İnek)** ve **Rasyon Ustası (Besi Sığırı)** uygulamalarının kullanıcı arayüzünden (UI) tamamen bağımsız, saf JavaScript/TypeScript bilimsel hesaplama çekirdeğidir.

---

## 1. "Süt Ustası" — Süt Sığırı Rasyonu Motoru (`dairy-engine.js`)

### 1.1 Bilimsel Standartlar
- **NRC 2021 (NASEM, 8th Revised Edition — *Nutrient Requirements of Dairy Cattle*):** Yaşama payı net laktasyon enerjisi ($\text{NE}_L$), süt üretim enerjisi, kuru madde tüketim kapasitesi ($\text{DMI}$), metabolize olabilir protein ($\text{MP}$), RUP/RDP dengesi ve kalsiyum/fosfor gereksinimleri.
- **INRA 2018 & CNCPS v6.5:** Fiziksel etkili lif (NDF), işkembe tamponlama ve laktasyon dönemine göre negatif enerji dengesi (NEB) kontrolü.
- **DCAD Denklemi (Dietary Cation-Anion Difference):** Kuru dönem ineklerinde doğum felci (süt humması / hipokalsemi) önleme hesabı:
  $$\text{DCAD (mEq/kg KM)} = (\text{Na} + \text{K}) - (\text{Cl} + \text{S})$$
- **TAGEM (Türkiye Yem Değerleri):** 42+ yerli kaba, kesif ve katkı yeminin kuru madde, $\text{NE}_L$, HP, RUP, NDF, ADF, Ca, P, K ve DCAD kompozisyonu (`feeds-db.json`).

### 1.2 NRC 2021 Temel Denklemleri (`nrc-2021-dairy-tables.json`)
1. **Yaşama Payı Enerjisi ($\text{NE}_{Lm}$):**
   $$\text{NE}_{Lm} \text{ (Mcal/gün)} = 0.080 \times \text{CA}^{0.75}$$
   *(Açık/sundurma barınmada soğuk stresi için $+\%12$ düzeltme uygulanır.)*
2. **Süt Enerji İhtiyacı ($\text{NE}_{Ll}$):**
   $$\text{NE}_L \text{ (Mcal / kg süt)} = 0.0929 \times \text{Yağ\%} + 0.0547 \times \text{Protein\%} + 0.192$$
   *(Süt yağı bilinmiyorsa varsayılan $\%3.7$, protein bilinmiyorsa $\%3.2$ alınır ve raporda "tahmini" olarak işaretlenir.)*
3. **Gebelik ve Büyüme Payı:**
   - Uzak kuru dönem (doğuma 60 gün): $+2.8\text{ Mcal NE}_L/\text{gün}$
   - Yakın kuru dönem (doğuma 21 gün): $+3.6\text{ Mcal NE}_L/\text{gün}$
   - 1. buzağılı genç inek büyüme payı: $+1.45\text{ Mcal NE}_L/\text{gün}$ ve $+\%0.6\text{ HP}$
   - 2. buzağılı inek büyüme payı: $+0.65\text{ Mcal NE}_L/\text{gün}$
4. **Vücut Kondisyon Skoru (VKS) Düzeltmesi:**
   - $1\text{ kg}$ canlı ağırlık değişimi $\approx 5.5\text{ Mcal NE}_L$
   - Zayıf ineklerde ($\text{VKS } 2.0 - 2.5$) günlük $+0.75\text{ ile }+1.35\text{ Mcal NE}_L$ eklenir.

### 1.3 6 Farklı Laktasyon Dönemi Mantığı ve Sert Kısıtlar (Hard Constraints)
| Kısıt | Kural | Bilimsel Gerekçe |
|---|---|---|
| **Geviş Lifi (NDF)** | $\ge \%28\text{ KM}$ (Sağmalda $\le \%40\text{ KM}$) | Süt yağının düşmesini ve işkembe asidozunu önler |
| **Kaba Yem Oranı** | Sağmalda $\ge \%40\text{ KM}$, Kuru dönemde $\ge \%70\text{ KM}$ | Geviş getirme ve abomasum deplasmanı önleme |
| **Tahıl (Arpa/Mısır) Sınırı** | Kesif yem içinde $\le \%45$ | İşkembe ekşimesi ve laminitis (topallık) önleme |
| **Korunmuş Yağ** | Toplam KM içinde $\le \%5$ ($\sim 0.30\text{ kg/gün}$) | Rumen lif sindirimini bozmadan enerji sağlama |
| **Yemlik Üre** | Toplam KM içinde $\le \%0.5$ | Amonyak zehirlenmesi ve döl tutmama önleme |
| **Ca : P Oranı** | Sağmalda $1.5:1 - 2.5:1$ | Sütle atılan kalsiyumun dengelenmesi |
| **Kuru Dönem DCAD** | $-100\text{ ile }+200\text{ mEq/kg KM}$ (Yakın kuru / süt humması geçmişi) | Paratiroid hormonunu aktif tutarak süt hummasını önleme |

---

## 2. "Rasyon Ustası" — Besi Sığırı Rasyonu Motoru (`beef-engine.js`)

- **NRC 2016 (*Nutrient Requirements of Beef Cattle*, 8. Baskı) & INRA 2018:** `nrc-2016-tables.json` tablo interpolasyonu.
- **Sert Kısıtlar:** Kaba yem $\ge \%30\text{ KM}$, NDF $\ge \%25\text{ KM}$, kesif yem içinde Arpa $\le \%50$, yemlik üre $\le \%1\text{ KM}$ (0–6 ay yasak), $\text{Ca:P } 1.2 - 2.5$.

---

## 3. "Kuzu Ustası" — Besi Kuzusu & Koyun Rasyonu Motoru (`sheep-engine.js`)

### 3.1 Bilimsel Standartlar (`nrc-2007-sheep-tables.json`)
- **NRC 2007 (*Nutrient Requirements of Small Ruminants: Sheep, Goats, Cervids*) + INRA 2018 + AFRC 1993 + CSIRO 2007:**
  - **Yaşama Payı Enerjisi ($\text{ME}_m$):** $\text{ME}_m = 0.056 \times \text{SBW}^{0.75}$ (+ yapağı/hareket payı, açık ağılda soğuk stresi $+\%12$, merada $+\%15$).
  - **Büyüme (ADG) ve Kuru Madde ($\text{DMI}$):** Canlı ağırlık ($15–70\text{ kg}$) ve günlük canlı ağırlık artışı ($100–350\text{ g/gün}$) 2 boyutlu NRC 2007 tablo interpolasyonu.
  - **Anaç Koyun Gebelik & Laktasyon:** Gebeliğin son 2 ayında enerji $+\%30$ (ikizde $+\%45$), protein $+\%40$; emziren koyunda süt enerjisi $1.35\text{ Mcal ME/kg süt}$.

### 3.2 Koyuna Özel Görünmez Zorunlu Kısıtlar (Hard Constraints)
1. **Bakır Zehirlenmesi Koruması (Copper Toxicity):**
   - Koyunlar bakıra sığırdan 10 kat daha duyarlıdır. Rasyon kuru maddesinde **Bakır ($\text{Cu}$) asla $15\text{ mg/kg KM}$ sınırını geçemez** (toksik eşik: $25\text{ mg/kg KM}$).
   - **Molibden ($\text{Mo}$) $\ge 0.5\text{ mg/kg KM}$** tutularak bakır-molibden-kükürt dengesi korunur.
   - **Sığır mineral premiksi (`sigir_premiksi`) seçilirse sistem kullanıcıyı uyarır, seçimi engeller ve bakırsız `koyun_premiksi` ile değiştirir.**
2. **İdrar Taşı (Urinary Calculi) Koruması (Erkek ve Karışık Kuzularda Zorunlu):**
   - $\text{Ca:P} \ge 2.0:1$ ($2.0 - 3.0$ aralığında mermer tozu / yonca ile dengelenir)
   - $\text{Tuz} \ge \%0.5\text{ KM}$ (hedef $\sim \%0.8\text{ KM}$, su tüketimini artırmak için)
   - $\text{Amonyum Klorür} \ge \%0.5\text{ KM}$ (idrar asitleyici olarak otomatik eklenir)
3. **Rumen ve Pamuk Küspesi (Gossypol) Kısıtları:**
   - $\text{NDF } \%25 - \%45\text{ KM}$, Kaba yem $\ge \%30\text{ KM}$ (Tokluda $\%50 - \%60\text{ KM}$)
   - Pamuk tohumu küspesi $\le \%10\text{ KM}$ (gossypol sınırı), 0–6 ay kuzularda yemlik üre yasak.

---

## 4. Birim Testleri (86 Test)

```bash
node --test lib/ration-engine/sheep-engine.test.mjs lib/ration-engine/dairy-engine.test.mjs lib/ration-engine/beef-engine.test.mjs
```
- `sheep-engine.test.mjs`: **32 birim testi** (NRC 2007 $\pm\%5$ doğrulaması, bakır $\le 15\text{ mg/kg KM}$, erkek kuzu idrar taşı koruması, sığır premiksi engeli, gebe/emziren anaç koyun ve 5 örnek senaryo).
- `dairy-engine.test.mjs`: **32 birim testi** (NRC 2021 $\pm\%5$ doğrulaması, 5 örnek senaryo, kuru dönem DCAD, yeni doğum ketozis ve tüm sert kısıtlar).
- `beef-engine.test.mjs`: **22 birim testi** (NRC 2016 $\pm\%5$ doğrulaması ve besi kısıtları).

