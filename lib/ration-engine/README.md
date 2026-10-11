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

## 3. Birim Testleri (54 Test)

```bash
node --test lib/ration-engine/dairy-engine.test.mjs lib/ration-engine/beef-engine.test.mjs
```
- `dairy-engine.test.mjs`: **32 birim testi** (NRC 2021 $\pm\%5$ doğrulaması, 5 örnek senaryo, kuru dönem DCAD, yeni doğum ketozis ve tüm sert kısıtlar).
- `beef-engine.test.mjs`: **22 birim testi** (NRC 2016 $\pm\%5$ doğrulaması ve besi kısıtları).
