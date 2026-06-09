# Fish Farm — Oyun Tasarım Belgesi

**Platform:** www.clickingame.com  
**Teknoloji:** Vanilla HTML/CSS/JavaScript (framework yok)  
**Dosya:** `fish-farm/index.html` (~2,451 satır, tek dosya)

---

## Oyun Özeti

Deniz altı bir imparatorluk kur. Balık tıklayarak, binalar inşa ederek ve yükseltmeler satın alarak üretimini katlayıp okyanusun efendisi ol.

---

## Para Birimi ve İlerleme

**Ana para birimi:** Balık (🐟)

Toplam kazanılan balıka göre tıklama butonu evrimi:

| Eşik | Aşama | Emoji |
|------|-------|-------|
| 0 | Baby Fish | 🐟 |
| 500 | Tropical Fish | 🐠 |
| 5,000 | Puffer Fish | 🐡 |
| 50,000 | Shark | 🦈 |
| 500,000 | Dolphin | 🐬 |
| 5,000,000 | Blue Whale | 🐳 |
| 50,000,000 | Sea Overlord | 🦭 |
| 500,000,000 | Ocean Dragon | 🐲 |

---

## Tıklama Sistemi

- Temel tıklama gücü: 1 balık
- Çarpan yükseltmeleri ile üstel büyüme
- **Click Synergy:** Her tıklamada balık/sn üretiminin %1-30'u kadar ek kazanç
- **Bot Koruması:** 2 saniyede max 30 tıklama; aynı noktaya max 15 ardışık tıklama; ihlalde 4s dondurma

---

## Binalar (18 adet)

Her bina otomatik olarak balık/sn üretir. Maliyet her alımda **1.28x** artar.

| # | Bina | Balık/sn |
|---|------|----------|
| 1 | Fishing Rod | 0.08 |
| 2 | Fisher Granny | 0.5 |
| 3 | Boat | 2 |
| 4 | Dock | 8 |
| 5 | Fish Farm | 25 |
| 6 | Hatchery | 75 |
| 7 | Trawler | 200 |
| 8 | Submarine | 500 |
| 9 | Sonar Station | 1,200 |
| 10 | Coral Reef Farm | 3,000 |
| 11 | Aquarium Lab | 7,500 |
| 12 | Processing Factory | 18,000 |
| 13 | Fish Satellite | 45,000 |
| 14 | Ocean Portal | 110,000 |
| 15 | Volcano Aquafarm | 280,000 |
| 16 | Time Machine | 700,000 |
| 17 | Black Hole Fisher | 1,800,000 |
| 18 | Poseidon's Palace | 60,000,000 |

Kilit açma koşulu: Önceki binalarda belirli miktarlar gerekir.  
Görsel: "Farm" sahnesinde sallanma animasyonlu ikonlar.

---

## Yükseltmeler (31 adet)

Her yükseltme yalnızca bir kez satın alınabilir.

### Tıklama Yükseltmeleri (6)
Sharp Hook → Titanium Hook → Golden Rod → Click Frenzy → Ancient Map → Kraken Claw

### Bina Güçlendiricileri (16)
Her binaya özel 2-4x çarpan veren yükseltmeler.

### Global Çarpanlar (3)
Tüm üretimi 1.5-3x artıran yükseltmeler.

### Sinerji Yükseltmeleri (4)
Her tıklamaya balık/sn'nin %1-30'unu ekler.

---

## Bonus Balık Sistemi

Her **18-40 saniyede** bir rastgele bonus balık belirir.

### Nadirlik Dağılımı

| Nadirlik | Olasılık | Animasyon |
|----------|----------|-----------|
| Common | %50 | Wobble |
| Rare | %30 | Bounce |
| Epic | %15 | Spiral |
| Legendary | %5 | Flash |

### Ödül Tipleri

| Tip | Etki |
|-----|------|
| Instant | Anlık balık kazancı (çarpan: 0.6-60) |
| Frenzy | Tıklama çarpanı (1.3-4x, 8-20s) |
| Harvest | Üretim çarpanı (1.3-4x, 8-30s) |

**23 farklı tür:** Clownfish, Lobster, Shark, Dolphin, Blue Whale, Poseidon's Gift, vb.

---

## Kriz Sistemi

**Tetiklenme:** 3+ bina sahibi olununca; 45-120s aralıklarla.  
Çözülmezse üretimde ceza uygulanır.

### Kriz Türleri

| Kriz | Etkilenen Bina | Çözüm Tipi | Tıklama | Ceza |
|------|---------------|-----------|---------|------|
| Storm | Boats | Mash | 35 | -%35 üretim |
| Pollution | Factory | Rapid | 55 | -%20 üretim |
| Shipwreck | Trawler | Click | 70 | -%25 üretim |
| Disease | Farm | Hold | 50 | -%15 üretim |
| Poachers | Boats | Mash | 42 | -%45 üretim |
| Algae Bloom | Reef | Rapid | 63 | -%25 üretim |
| Tsunami | Docks | Click | 84 | -%10 üretim |
| Sonar Glitch | Sonar | Hold | 42 | -%30 üretim |
| Lab Fire | Lab | Mash | 63 | -%20 üretim |
| Hatchery Flood | Hatchery | Rapid | 49 | -%25 üretim |

### Çözüm Mekanikleri
- **Mash:** Her eylemde 2 tıklama düşer
- **Click:** Her eylemde 1 tıklama düşer
- **Rapid:** Her eylemde 1 tıklama düşer (hızlı)
- **Hold:** Buton basılı tutulurken 120ms aralıklarla sayar

---

## UI Düzeni (3-Sütun Grid)

```
[ Yükseltmeler + İstatistikler ] | [ Balık Sayacı + Tıklama + Farm Sahnesi ] | [ Binalar ]
```

---

## İstatistikler (8 metrik)

1. Toplam kazanılan balık
2. Yakalanan bonus balık sayısı
3. Toplam tıklama sayısı
4. Tıklamadan kazanılan balık
5. Toplam bina sayısı
6. Toplam yükseltme sayısı
7. İlk oynanma tarihi
8. Toplam oyun süresi

---

## Kayıt Sistemi

- Her **10 saniyede** otomatik kayıt
- Sayfa kapanırken kayıt
- `localStorage` anahtarı: `fishfarm_save`
- Bozuk kayıt tespiti ve yönetimi

---

## Özel Teknik Özellikler

| Özellik | Açıklama |
|---------|----------|
| Tank Canvas | Tıklama alanında animasyonlu arkaplan |
| Parçacık Efektleri | Tıklamada yükselen sayı animasyonları |
| Milestone Banner | Altın gradyan başarı bildirimleri |
| Paylaşım | html2canvas ile ekran görüntüsü |
| Kriz Efekti | Etkilenen bina satırı kırmızı vignette ile vurgulanır |
| Aktif Boost Barları | Aktif çarpanlar için alt-merkez ilerleme çubukları |

---

## İlerleme Eğrisi

| Aşama | Balık Aralığı | Oynanış |
|-------|--------------|---------|
| Erken | 0 – 1,000 | Manuel tıklama, ilk 1-2 bina |
| Orta | 1K – 100K | 6-8 bina, başlangıç yükseltmeleri |
| Geç | 100K – 50M | Tüm binalar, tam yükseltme ağacı |
| Sonlu | 50M+ | Optimizasyon, idle ilerleme |

---

## Kullanılan Kütüphaneler

- **Google Fonts:** Fredoka One (başlıklar), Nunito (gövde)
- **html2canvas v1.4.1:** Ekran görüntüsü paylaşımı
- **Google AdSense:** Reklam sistemi
