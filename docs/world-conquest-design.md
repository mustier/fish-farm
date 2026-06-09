# World Conquest — Oyun Tasarım Belgesi

**Proje Adı:** World Conquest  
**Platform:** clickingame.com  
**Teknoloji:** Vanilla HTML/CSS/JS — tek dosya mimarisi  
**Dosya:** `world-conquest/index.html` (~2600+ satır)  
**Tahmini Oyun Süresi:** 8-20 saat (birinci oynanış), New Game+ ile tekrar oynanabilir

---

## Konsept Özeti

Tek bir başkent bölgesi ile başlıyorsun. Tıklayarak kaynak üretir, yapılar inşa eder, askeri kuvvetler oluşturur ve dünyanın tüm bölgelerini ele geçirirsin. Her bölge benzersiz kaynaklar barındırır, bazıları düşman ülkelerin kontrolündedir, bazıları boştur. Tüm dünya ele geçirildikten sonra uzay savaşları ve uzaylı istilası başlar.

---

## 1. KAYNAK SİSTEMİ

### 1.1 Temel Kaynaklar (her zaman üretilebilir)
| Kaynak | Ikon | Açıklama |
|--------|------|----------|
| Altın | 💰 | Evrensel para birimi; bina, asker, araştırma |
| Nüfus | 👥 | Çalışan, asker kaynağı; büyüme organik |
| Yiyecek | 🌾 | Nüfus büyümesi ve ordunun iaşesi |
| Enerji | ⚡ | Fabrika ve ileri yapılar için |
| Demir | ⚙️ | Kara ordusu, binalar — primary kaynak olarak gösterilir |
| Petrol | 🛢️ | Hava & deniz kuvvetleri — primary kaynak |
| Ahşap | 🪵 | Erken yapılar, ticaret gemisi — primary kaynak |
| Bakır | 🔶 | Elektronik altyapı — primary kaynak |

> **Önemli:** `primary` dizisi `['gold','food','energy','pop','iron','oil','wood','copper']` — bu kaynaklar kaynak çubuğunda HER ZAMAN gösterilir, değer 0 olsa bile. `secondary` dizisindeki kaynaklar yalnızca üretiliyorken görünür.

### 1.2 Bölgeye Özgü Kaynaklar (secondary)
| Kaynak | Ikon | Bölge Tipi | Kullanım |
|--------|------|-----------|----------|
| Silisyum | 💎 | Teknoloji bölgesi | Elektronik, uzay |
| Uranyum | ☢️ | Sarp dağlar | Nükleer, uzay silahları |
| Gıda Fazlası | 🌽 | Verimli ova | Ordunun hızlı toparlanması |
| Elmas | 💍 | Afrika/Güney bölge | Diplomatik güç, lüks yükseltmeler |
| Nadir Toprak | 🌐 | Asya iç bölge | Uzay teknolojisi |
| Buz Çekirdeği | 🧊 | Kutup | Uzay-sonrası içerik |

---

## 2. HARİTA & BÖLGE SİSTEMİ

### 2.1 SVG Harita Mimarisi

- **viewBox:** `0 0 1100 580`
- **Bölge sayısı:** 38 (SVG polygon + label)
- **Shared-edge prensibi:** Komşu bölgeler sınırlarındaki köşe noktalarını tam olarak paylaşır. Örnek: `western_europe` ve `northern_europe` `354,136` ve `380,150` noktalarını her ikisi de içerir.
- **Grid snap:** Tüm koordinatlar ~22px grid'e yaslandı — koordinat kaymaları önlenir.
- **Kıta renklendirme:** Her bölgeye `cont-[kıta]` CSS sınıfı verilir → farklı tonlar

**CONTINENT_CLASS haritası (JavaScript):**
```js
const CONTINENT_CLASS = {
  british_isles:'europe', scandinavia:'europe', northern_europe:'europe',
  western_europe:'europe', iberia:'europe', eastern_europe:'europe', balkans:'europe',
  russia:'russia', siberia:'russia',
  caucasus:'middleeast', middle_east:'middleeast', arabian_peninsula:'middleeast',
  central_asia:'asia', mongolia:'asia', china:'asia', japan_korea:'asia',
  india:'asia', southeast_asia:'asia',
  north_africa:'africa', west_africa:'africa', central_africa:'africa',
  east_africa:'africa', south_africa:'africa',
  north_america_west:'americas', north_america_east:'americas', central_america:'americas',
  caribbean:'americas', south_america_north:'americas', amazon:'americas',
  south_america_east:'americas', south_america_west:'americas',
  australia:'oceania', new_zealand:'oceania', pacific_islands:'oceania',
  greenland:'polar', arctic:'polar',
};
```

**Kıta CSS renkleri:**
```css
.region.cont-europe    { fill:#2a4a7a; }
.region.cont-russia    { fill:#1e3d5c; }
.region.cont-asia      { fill:#3a2a5e; }
.region.cont-middleeast{ fill:#5a3a18; }
.region.cont-africa    { fill:#5c3318; }
.region.cont-americas  { fill:#1a4a2e; }
.region.cont-oceania   { fill:#1a3a4e; }
.region.cont-polar     { fill:#1e2e4a; }
```

**SVG render CSS (önemli):**
```css
.region {
  stroke: rgba(255,255,255,0.25);
  stroke-width: 1.5;
  stroke-linejoin: round;
  paint-order: stroke fill;  /* border üstüne fill değil, fill üstüne border */
}
```

### 2.2 Bölge Tipleri
| Tip | İkon | Özellik |
|-----|------|---------|
| Ova | 🟩 | Yüksek nüfus, orta kaynak |
| Dağlık | 🏔️ | Demir/uranyum, zor savunma — kara saldırıya direnç +30% |
| Kıyı | 🌊 | Deniz yolu avantajı, petrol |
| Çöl | 🏜️ | Petrol, düşük nüfus |
| Orman | 🌲 | Ahşap, gizlilik |
| Teknoloji | 🏙️ | Silisyum, araştırma hızı |
| Kutup | 🧊 | Buz çekirdeği — tüm bakım maliyeti +40% |

### 2.3 Bölge Durumu
- **Boş:** Sahipsiz. Düşük direniş ama isyancı riski var.
- **Kendi kontrolünde:** Tıklamayla kaynak üretir, yapı kurulabilir.
- **Düşman ülke:** İşgal edilmeden üretilemeyen kaynaklar.

### 2.4 Kolonizasyon Değişikliği
**Eski tasarım:** Boş bölgeler anında altın ödeyerek alınıyordu.  
**Yeni tasarım:** `colonize()` fonksiyonu artık `openCombat()` çağırıyor — tüm bölge alımları savaş gerektiriyor.

```js
function colonize(regionId) {
  openCombat(regionId);
}
```

---

## 3. SAVAŞ SİSTEMİ (İmplementasyon Detayları)

### 3.1 İlk Savaş Gösterimi
`state.firstCombatShown` flag'i ile ilk savaşta tutorial gösterilir:

```html
<div id="combat-howto" style="display:none; background:rgba(255,255,150,0.15);
  border:1px solid #ffd700; border-radius:8px; padding:10px; margin-bottom:10px; font-size:0.85em;">
  <b>⚔️ How Combat Works</b><br>
  • Rounds auto-fire every 2 seconds — watch the progress bars<br>
  • Click during combat to deal extra damage each round<br>
  • Units are permanently lost when you take damage<br>
  • You can retreat anytime to save remaining forces<br>
  • Win to capture the region — lose and it stays enemy
</div>
```

### 3.2 Kayıp Takibi (preForce Snapshot)
Savaş başlarken anlık snapshot alınır:

```js
// startCombatWithForce içinde:
const preForce = {};
Object.keys(force).forEach(uid => { preForce[uid] = state.mil[uid] || 0; });
state.combat.preForce = preForce;
```

### 3.3 Savaş Özeti (showCombatSummary)
```js
function showCombatSummary(won) {
  const c = state.combat;
  const casualties = {};
  if (c.preForce) {
    Object.keys(c.preForce).forEach(uid => {
      const lost = c.preForce[uid] - (state.mil[uid] || 0);
      if (lost > 0) casualties[uid] = lost;
    });
  }
  // yeşil = zafer kutusu, kırmızı = yenilgi kutusu
  // kayıp listesini gösterir
  document.getElementById('retreat-btn').textContent = '✕ Close';
}
```

### 3.4 Retreat → Close Dönüşümü
```js
function retreatOrClose() {
  if (state.combat && state.combat.done) {
    closeCombatModal();
  } else {
    retreatCombat();
  }
}
```

Buton `onclick="retreatOrClose()"` ile bağlı. Savaş bitince text `✕ Close` olur.

### 3.5 Kayıplar Kalıcıdır
Birimler global `state.mil` üzerinden takip ediliyor. Savaştaki hasar direkt `state.mil[uid]` değerini azaltıyor. `preForce` snapshot ile sadece gösterim için önceki değer hatırlanıyor, sıfırlanmıyor.

---

## 4. YÜKSELTME AĞACI (TECH TREE)

### 4.1 Yapı
- 3 dal: `economy`, `science`, `military`
- 83 araştırma (başlangıçta 41'den genişletildi)
- DFS sıralaması ile `buildTechTreeOrder()` — ön gereksinimler önce gelir
- Her araştırmanın `req:[]` dizisi önceki araştırmaları listeler

### 4.2 Ekonomi Dalı (örnek)
```js
{ id:'agriculture', name:'Agriculture', icon:'🌾', cost:{gold:50}, req:[], branch:'economy' },
{ id:'advanced_farming', name:'Advanced Farming', icon:'🚜', cost:{gold:150,food:50}, req:['agriculture'] },
{ id:'industrial_farm', name:'Industrial Farm', icon:'🏭', cost:{gold:500,food:200}, req:['advanced_farming'] },
// ... devam eder
```

### 4.3 Tech Düzenleme Notu
TECH_DEFS dizisi emoji karakterleri içerdiğinden Edit tool'u ile düzenlemek başarısız oluyor.  
**Çözüm:** Python line-range replacement kullanıldı:
```python
with open('world-conquest/index.html', 'r') as f:
    lines = f.readlines()
lines[start:end] = new_content
with open('world-conquest/index.html', 'w') as f:
    f.writelines(lines)
```

---

## 5. YAPI SİSTEMİ (BUILDING_DEFS)

### 5.1 Village Bug Fix
**Problem:** Village `terrains:['plains','coast','forest']` ile tanımlanmıştı. Haritadaki çoğu bölge `tech`, `mountains`, `desert`, `polar` terrain'e sahip olduğundan village hiçbir yerde inşa edilemiyordu.  
**Çözüm:** `terrains:[]` — tüm terrain tiplerinde inşa edilebilir.

```js
{ id:'village', name:'Village', icon:'🏘️', desc:'+0.5 pop/s; +50 pop cap',
  cost:{gold:100}, terrains:[], req_research:['agriculture'] },
```

### 5.2 Bölgeye Özgü Yapılar
`terrains:['coast']`, `terrains:['mountains']` gibi kısıtlamalar sadece gerçekten arazi-bağımlı yapılarda kullanılır (petrol kuyusu, demir madeni, liman vb.).

---

## 6. KAYNAK HEADER BARI

### 6.1 Konum
Kaynak bar (`#res-strip`) haritanın üstünde, ana header'ın altında konumlandırılmıştır:

```html
<section class="map-area">
  <div id="res-strip">
    <div class="res-bar" id="res-bar"></div>
  </div>
  <div class="map-header">...</div>
  <div class="map-wrap">...</div>
</section>
```

### 6.2 Gösterim Mantığı
```js
const primary   = ['gold','food','energy','pop','iron','oil','wood','copper'];
const secondary = ['silicon','uranium','surplus_food','diamond','rare_earth','ice_core'];

// primary: her zaman göster
// secondary: rate > 0.01 veya değer > 0 ise göster
```

---

## 7. UI DÜZENİ (3-SÜTUN GRID)

```
┌─────────────────────────────────────────────────────────────┐
│ [🌍 LOGO]     Başlık / Ana Header                           │ ← Header
├─────────────────────────────────────────────────────────────┤
│  ← Kaynak Bar (gold, food, energy, pop, iron, oil, wood...) │ ← #res-strip
├────────────────┬──────────────────────────┬─────────────────┤
│                │   Harita Header           │                 │
│  TEKNOLOJİ     │   (bölge bilgisi)         │  ASKERİ PANEL   │
│  AĞACI         │                           │                 │
│                │   🗺️ SVG HARİTA            │                 │
│  [Ekonomi]     │   38 bölge, kıta renkleri │ Birim listesi   │
│  [Bilim]       │   Tıklanabilir bölgeler   │ Savaş butonları │
│  [Askeri]      │                           │                 │
└────────────────┴──────────────────────────┴─────────────────┘
```

---

## 8. TEKNİK NOTLAR & HATALAR

### 8.1 Edit Tool ile Emoji Sorunu
TECH_DEFS, BUILDING_DEFS, REGIONS gibi emoji içeren bloklar Edit tool ile düzenlenmek istendiğinde Unicode mismatch hatası veriyor. **Çözüm:** Python scripti ile line number üzerinden değiştirme.

### 8.2 Python Script ile SVG Grid Hatası
SVG grid/background bloğunu Python ile değiştirirken kazara çift kopyalanma ve yanlış konumlandırma oldu. **Ders:** SVG bloklarını değiştirirken eski bloğu önce tamamen sil, sonra yeni bloğu ekle.

### 8.3 renderMapColors() Dikkat
Bu fonksiyon çağrıldığında bölge SVG elemanlarından `class` attribute'unu tamamen sıfırlıyor. Bu yüzden kıta CSS sınıflarını (`cont-europe` vs) burada yeniden uygulamak gerekiyor:

```js
function renderMapColors() {
  REGIONS.forEach(r => {
    const el = document.getElementById('r_' + r.id);
    if (!el) return;
    const contClass = CONTINENT_CLASS[r.id] ? ' cont-' + CONTINENT_CLASS[r.id] : '';
    // ... owner/selected renklendirmesi + contClass ekle
  });
}
```

### 8.4 Force Azalması Sorusu
Kullanıcı "Neden kuvvetlerim zamanla azalıyor?" diye sordu. **Cevap:** Kuvvetler azalmıyor, kaynak çubuğundaki sayılar kaynak üretimine göre değişiyor. Ancak bakım maliyeti (maintenance cost) varsa `state.mil` değil `state.res.gold` azalır.

---

## 9. KAYNAK SİSTEMİ DETAYLARI

### 9.1 Kayıt Sistemi
- localStorage anahtarı: `worldconquest_save`
- Her 15s otomatik kayıt + sayfa kapanırken kayıt

### 9.2 State Nesnesi (Özet)
```js
state = {
  res: { gold, food, energy, pop, iron, oil, wood, copper,
         silicon, uranium, surplus_food, diamond, rare_earth, ice_core },
  regions: { [id]: { owner, buildings[], defPower, terrain, specialRes[], neighbors[] } },
  mil: { militia, infantry, sniper, tank, ... },  // global, bölge bağımsız
  research: Set(completedIds),
  combat: { active, regionId, turns, attackForce, defForce, preForce, done },
  firstCombatShown: bool,
  stats: { totalClicks, regionsConquered, unitsLost, ... },
  totalTime, startedAt
}
```

---

## 10. OYUN AKIŞI & AŞAMALAR

### Aşama 1 — KURULUŞ (0-30 dk)
Başlangıç bölgesi → ilk komşu boş bölgeleri ele geçir → Demir bölgesi → Köy + Kışla

### Aşama 2 — GENIŞLEME (30 dk - 2 saat)
Petrol → Hava kuvvetleri; Kıyı → Deniz kuvvetleri; İlk düşman ülkeyle karşılaşma

### Aşama 3 — GÜÇ SAVAŞI (2-6 saat)
Dünya %50; Uranyum → Nükleer; Büyük Koalisyon tehdidi

### Aşama 4 — DÜNYA HAKİMİYETİ (6-12 saat)
3 Boss Ülke; Nadir Toprak + Buz Çekirdeği; Uzay İstasyonu

### Aşama 5 — UZAY ÇAĞI (12-20 saat)
Uzaylı dalgaları; Ay, Mars, Asteroid; Nihai zafer

---

## 11. ASKERİYE SİSTEMİ (Tasarım)

Detaylı birim tabloları önceki tasarım bölümünde bulunabilir.  
**Mevcut implemente:**
- `state.mil` üzerinde global birim takibi
- Savaşta `preForce` snapshot ile kayıp görüntüleme
- Kuvvet avantaj/dezavantaj matrisi (tasarım aşamasında)

---

## 12. YÜKSELTME AĞACI — DAL YAPISI

### DAL A: EKONOMİ
```
Tarım I → Tarım II → Endüstriyel Çiftlik → Nano Tarım
    └─── Ticaret Yolu → Küresel Pazar → Uzay Ticareti
              └─── Tedarik Ağı → Lojistik İmparatorluğu

Madencilik I → Derin Madencilik → Nano Madencilik
    └─── Enerji Santrali → Nükleer Enerji → Füzyon Reaktörü
```

### DAL B: TEKNOLOJİ & ARAŞTIRMA
```
Temel Araştırma → Üniversite → Araştırma Merkezi → Küresel Ar-Ge
    └─── Elektronik → Bilgisayar → Yapay Zeka → Kuantum Ağı
              └─── İletişim → Uydu Ağı → Uzay İstasyonu
                        └─── Lazer Silahı → Yörünge Topu → Uzay Kalesi
```

### DAL C: ASKERİYE
```
Kara Doktrini I → Kara Doktrini II → Zırhlı Savaş → Nükleer Kara
Deniz Doktrini I → Deniz Doktrini II → Okyanus Hâkimiyeti → Nükleer Deniz
Hava Doktrini I → Hava Doktrini II → Hava Üstünlüğü → Orbital Saldırı
Uzay Doktrini I → Uzay Doktrini II → Galaktik Savaş  (dünya sonrası)
```

---

## 13. GELİŞTİRME SIRASI (MVP → Full)

### MVP (İlk Sürüm) ✅ Tamamlandı
- [x] SVG dünya haritası (38 bölge, kıta renkleri)
- [x] 8 temel kaynak + 6 özel kaynak
- [x] Kara kuvvetleri + basit savaş mekaniği
- [x] Temel upgrade tree (83 araştırma, 3 dal)
- [x] Savaş kayıp takibi + özet ekranı
- [x] İlk savaş tutorial metni
- [x] Kolonizasyon → savaş gerektiriyor
- [x] Village bug fix (terrains:[])

### v1.0 (Sonraki)
- [ ] Düşman AI (5 tip + Büyük Koalisyon)
- [ ] Ticaret & tedarik hattı sistemi
- [ ] Tüm yapılar (bölgeye özgü)
- [ ] 4 askeri dal tam birimler + avantaj matrisi
- [ ] Kuvvet avantaj/dezavantaj matrisi savaşa entegre

### v1.5
- [ ] Uzay aşaması + uzaylı dalgaları
- [ ] 12 gizli olay
- [ ] Savaş sinerjisi bonusları
- [ ] Diplomatik seçenekler
- [ ] New Game+ modu
- [ ] Detaylı istatistik & zafer ekranı

---

## 14. ARAŞTIRMA BULGULARI — HARİTA TASARIMI

Harita tasarımı için benzer oyunlardan alınan dersler:

1. **Shared-edge prensibi:** Risk, Civilization, Europa Universalis gibi oyunlarda bölgeler birbirine tam yaslanır — aralarında boşluk olmaz. SVG polygon'larda bu, komşu bölgelerin sınır noktalarını birebir aynı koordinatlarla paylaşmasıyla sağlanır.

2. **Kıta renk kodlaması:** En yüksek görsel ROI. Farklı kıtalar farklı mavi/mor/yeşil tonlarında olduğunda kullanıcı haritayı sezgisel okur.

3. **Grid snap:** ~22px aralıklı grid'e yaslamak kayma ve yanlış hizalamayı önler.

4. **Label konumları:** Bölge ismi etiketleri `(x, y)` merkezine konur, bölge şekli ne kadar garip olursa olsun. Label `pointer-events:none` ile tıklamayı geçirmeli.

5. **Stroke sırası:** `paint-order: stroke fill` CSS kuralı ile sınır çizgileri fill üstünde görünür, komşu bölgeler arasında net ayrım oluşur.
