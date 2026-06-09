# Mine Rush — Oyun Tasarım Belgesi

**Platform:** www.clickingame.com  
**Teknoloji:** Vanilla HTML/CSS/JavaScript (framework yok)  
**Dosya:** `mine-rush/index.html` (~1,820 satır, tek dosya)

---

## Oyun Özeti

Yerin derinliklerine in, nadir cevherler çıkar ve madencilik operasyonlarını otomatikleştir. Fish Farm ile paralel mekanik yapıya sahip, maden temalı idle/clicker oyunu.

---

## Para Birimi ve İlerleme

**Ana para birimi:** Cevher (⛏️)

Toplam kazanılan cevhere göre tıklama butonu evrimi:

| Eşik | Aşama |
|------|-------|
| 0 | Rusty Pickaxe (⛏️) |
| 500 | Stone Chunk |
| 5,000 | Iron Vein |
| 50,000 | Gold Nugget |
| 500,000 | Diamond Seam |
| 5,000,000 | Crystal Core |
| 50,000,000 | Star Metal |
| 500,000,000 | Volcanic Forge |

---

## Tıklama Sistemi

- Temel tıklama gücü: 1 cevher
- Çarpan yükseltmeleri ile üstel büyüme
- **Click Synergy:** Her tıklamada cevher/sn üretiminin belirli yüzdesi kadar ek kazanç
- Fish Farm ile aynı bot koruma sistemi (30 tıklama/2s limiti)

---

## Binalar (18 adet)

Her bina otomatik olarak cevher/sn üretir. Maliyet her alımda **1.28x** artar.

| # | Bina | Kategori |
|---|------|---------|
| 1 | Pickaxe | Erken |
| 2 | Miner | Erken |
| 3 | Ore Cart | Erken |
| 4 | Mine Shaft | Erken |
| 5 | Power Drill | Erken |
| 6 | Ore Smelter | Orta |
| 7 | Blasting Crew | Orta |
| 8 | Excavator | Orta |
| 9 | Ore Refinery | Orta |
| 10 | Geo Scanner | Orta |
| 11 | Geology Lab | Geç |
| 12 | Deep Core Drill | Geç |
| 13 | Mining Satellite | Geç |
| 14 | Asteroid Miner | Geç |
| 15 | Volcano Tap | Son aşama |
| 16 | Time Borer | Son aşama |
| 17 | Ore Singularity | Son aşama |
| 18 | Earth God's Vein | Son aşama |

---

## Yükseltmeler (31 adet)

Fish Farm ile paralel yapı:

### Tıklama Yükseltmeleri (6)
Sharp Pickaxe → Iron Pickaxe → altın ve üstü çeşitli kazma yükseltmeleri

### Bina Güçlendiricileri (16)
Her binaya özel 2-4x çarpan.

### Global Çarpanlar (3)
Tüm üretimi 1.5-3x artırır.

### Sinerji Yükseltmeleri (4)
Tıklama başına cevher/sn'nin %1-30'unu ekler.

---

## Bonus Cevher Sistemi

Her **18-40 saniyede** bir rastgele bonus cevher belirir.

### Nadirlik Dağılımı

| Nadirlik | Olasılık |
|----------|----------|
| Common | %50 |
| Rare | %30 |
| Epic | %15 |
| Legendary | %5 |

### Cevher Türleri (19 adet)
Stone Chunk, Copper Ore, Iron Vein, Gold Nugget, Diamond, Mythril, ve daha fazlası.

### Ödül Tipleri

| Tip | Etki |
|-----|------|
| Instant | Anlık cevher kazancı |
| Frenzy | Tıklama çarpanı (1.3-4x, 8-20s) |
| Harvest | Üretim çarpanı (1.3-4x, 8-30s) |

---

## Kriz Sistemi (10 kriz)

Madene özgü kriz olayları; Fish Farm ile aynı çözüm mekanikleri (Mash / Click / Rapid / Hold).

| Kriz | Açıklama |
|------|----------|
| Cave-In | Tünel çökmesi |
| Gas Leak | Gaz sızıntısı |
| Tunnel Flood | Tünel su basması |
| Equipment Failure | Ekipman arızası |
| Ore Raiders | Cevher hırsızları |
| Seismic Event | Sismik olay |
| Premature Blast | Erken patlama |
| Power Cut | Elektrik kesintisi |
| Smelter Fire | Eritme fırını yangını |
| Ore Shortage | Cevher kıtlığı |

---

## UI Düzeni (3-Sütun Grid)

```
[ Yükseltmeler + İstatistikler ] | [ Cevher Sayacı + Tıklama + Maden Sahnesi ] | [ Binalar ]
```

Fish Farm ile birebir aynı layout; tema ve renkler maden temasına uyarlanmış.

---

## Aktif Boost Barları

- Konum: Alt-merkez sabit
- Gösterim: Emoji, isim, çarpan değeri, geri sayım
- Nadirliğe göre süre değişir (8-30s)

---

## Reklam Entegrasyonu

- Sayfada Google AdSense banner slot bulunur
- Koşullu yükleme (iframe embed durumunda devre dışı)
- Placeholder ID: `ca-pub-XXXXXXXXXXXXXXXX`

---

## Kayıt Sistemi

- Her **10 saniyede** otomatik kayıt
- Sayfa kapanırken kayıt
- `localStorage` üzerinde saklanır
- Bozuk kayıt tespiti ve yönetimi

---

## Fish Farm ile Farklar

| Özellik | Fish Farm | Mine Rush |
|---------|-----------|-----------|
| Para Birimi | Balık 🐟 | Cevher ⛏️ |
| Tema | Okyanus / Deniz | Yeraltı / Maden |
| Bina Adları | Tekne, Çiftlik, Poseidon... | Kazma, Tünel, Asteroid... |
| Bonus Türleri | 23 deniz canlısı | 19 cevher türü |
| Kriz Sayısı | 11 | 10 |
| Satır sayısı | ~2,451 | ~1,820 |

---

## Kullanılan Kütüphaneler

- **Google Fonts:** Fredoka One (başlıklar), Nunito (gövde)
- **html2canvas v1.4.1:** Ekran görüntüsü paylaşımı
- **Google AdSense:** Reklam sistemi
