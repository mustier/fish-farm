# World Conquest — Oyun Tasarım Belgesi

**Proje Adı:** World Conquest  
**Platform:** clickingame.com  
**Teknoloji:** Vanilla HTML/CSS/JS — tek dosya mimarisi  
**Dosya:** `world-conquest/index.html` (~3074 satır)  
**Save Key:** `worldconquest_v1` (localStorage)  
**Auto-save:** 30 saniyede bir (`setInterval(autoSaveTimer_tick, 30000)`)  
**Tahmini Oyun Süresi:** 8-20 saat (birinci oynanış)

---

## 1. KAYNAK SİSTEMİ

### 1.1 Temel Kaynaklar (res-strip'te her zaman görünür)

| Kaynak | İkon | Başlangıç | Açıklama |
|--------|------|-----------|----------|
| Altın | 💰 | 300 | Evrensel para birimi |
| Yiyecek | 🌾 | 150 | Nüfus büyümesi + ordu iaşesi |
| Enerji | ⚡ | 80 | Fabrika ve ileri yapılar |
| Nüfus | 👥 | 60 | Asker ve çalışan kaynağı |
| Demir | ⚙️ | 0 | Kara ordusu + binalar |
| Petrol | 🛢️ | 0 | Hava & deniz kuvvetleri |
| Silisyum | 💡 | 0 | Elektronik + uzay |
| Uranyum | ☢️ | 0 | Nükleer + uzay silahları |
| Ahşap | 🪵 | 0 | Erken yapılar |
| Bakır | 🔧 | 0 | Elektronik altyapı |

### 1.2 Secondary Kaynaklar (üretilince görünür)

| Kaynak | İkon | Bölge Tipi |
|--------|------|-----------|
| Yiyecek Fazlası | 🌽 | Ova, kıyı |
| Elmas | 💎 | Afrika, Amazon |
| Nadir Toprak | 🔮 | Polar, Asya iç |
| Buz Çekirdeği | 🧊 | Kutup |

### 1.3 Kaynak Üretim Mantığı (computeRates)

- Her sahip olunan bölge: +2 altın/s, +0.5 yiyecek/s, +1 enerji/s, +0.1 nüfus/s (cap'e kadar)
- Bölge `specialRes` dizisindeki her kaynak: +1.5/s (araştırma çarpanlarıyla)
- Araştırma çarpanları: `ironMult`, `oilMult`, `silMult`, `goldMult`, `energyMult`, `woodMult`, `allProd`
- Boost sistemi: `state.activeBoosts[]` → `gold_mult`, `all_prod`, `uranium_mult` tipleri

### 1.4 Depo Kapları (computeStorageCaps)

Bölge sayısına göre dinamik olarak büyür:
- Altın: `1000 + landCount * 300`  
- Yiyecek: `500 + landCount * 150`  
- Enerji: `400 + landCount * 100`  
Terrain bonusları (dağ → demir+50, çöl → petrol+60, orman → ahşap+60, kutup → nadir_toprak+40).  
Bina bonusları (`warehouse` → tüm kaynaklar +500).  
Araştırma bonusları: `banking` → altın +2000, `quantum` → tüm kaplar ×1.3.

### 1.5 Bakım Maliyeti (applyUpkeep)

Her saniye `state.mil` üzerindeki birimlerden:
- Kara birimi: pop ×0.05/birim
- Deniz birimi: pop ×0.08/birim
- Hava birimi: pop ×0.06/birim
- Uzay birimi: pop ×0.12/birim
+ Birim bazlı yiyecek/petrol/enerji/uranyum upkeep'i

Kaynak sıfıra düşerse `desertion()` → rastgele ilgili birim silinir, bildirim gösterilir.

---

## 2. HARİTA & BÖLGE SİSTEMİ

### 2.1 SVG Harita Mimarisi

- **viewBox:** `-80 -10 1195 700`
- **Bölge sayısı:** 35 (REGIONS dizisi) — footer'da "0/35" yazıyor
- **Ocean background:** `radialGradient` `#0d2240` → `#060e1c`
- **Grid:** Latitude/longitude çizgileri (rgba opacity 0.05–0.08)
- **DOM:** `<g id="regions-g">` + `<g id="labels-g">` — buildMap() ile doldurulur

### 2.2 Bölgeler ve Kıtalar (tam liste)

| Kıta | Bölgeler |
|------|---------|
| Polar/Arktik | arctic, greenland, siberia_far_east, antarctica |
| Avrupa | scandinavia, british_isles, northern_europe, western_europe, iberia, eastern_europe, balkans |
| Rusya | russia |
| Kafkasya/Orta Asya | caucasus, central_asia |
| Orta Doğu | middle_east, arabian_peninsula |
| Afrika | north_africa, west_africa, central_africa, east_africa, south_africa |
| Asya | mongolia, china, japan_korea, india, southeast_asia |
| Amerika | north_america_west, north_america_east, central_america, caribbean, south_america_north, amazon, south_america_east, south_america_west |
| Okyanusya | pacific_islands, australia, new_zealand |

**Başlangıç bölgesi:** `western_europe` (tech terrain, specialRes: silicon + surplus_food)

### 2.3 Terrain Tipleri ve Savaş Modifikatörleri (TERRAIN_MOD)

| Terrain | Kara | Deniz | Hava | Uzay |
|---------|------|-------|------|------|
| plains | 1.0 | 0 | 1.0 | 1.0 |
| mountains | 0.7 | 0 | 1.1 | 1.0 |
| coast | 0.9 | 1.4 | 1.1 | 1.0 |
| desert | 0.8 | 0 | 1.0 | 1.0 |
| forest | 0.85 | 0 | 0.8 | 1.0 |
| tech | 1.0 | 1.0 | 1.2 | 1.3 |
| polar | 0.6 | 0.5 | 0.9 | 1.2 |

### 2.4 Bölge Durumu

- `neutral` — İşgal edilmemiş; `neutralDefense` değeri init'te rastgele atanır (`baseDefense * 0.2–0.5`)
- `player` — Oyuncuya ait; kaynak üretir, yapı kurulabilir
- `factionId` (militia/warlord/federation/empire/superpower/sea_empire/tech_nation) — Düşman faction
- `rebel` — Başkaldırı veya uzaylı istilası sırasında geçici durum

---

## 3. DÜŞMAN SİSTEMİ

### 3.1 Enemy Factions (ENEMY_FACTIONS)

| ID | İsim | Power Mult | Tutum | Tercih |
|----|------|-----------|-------|--------|
| militia | Rebels | 0.5 | aggressive | land |
| warlord | Warlord State | 0.8 | aggressive | land |
| federation | Federation | 1.0 | defensive | sea |
| empire | E. Empire | 1.3 | expansionist | air |
| superpower | Superpower | 1.8 | militarist | all |
| sea_empire | Sea Empire | 1.5 | expansionist | sea |
| tech_nation | Tech Nation | 2.0 | peaceful | air |

### 3.2 Başlangıç Atamaları (assignEnemies)

- Militia: west_africa, amazon, central_africa  
- Warlord: arabian_peninsula, mongolia, caucasus  
- Federation: north_america_east, north_america_west, central_america  
- Empire: china, southeast_asia, mongolia  
- Superpower: russia, eastern_europe  
- Sea Empire: japan_korea, pacific_islands, new_zealand  
- Tech Nation: india, south_america_north, east_africa

### 3.3 Enemy Tick (enemyTick — her 45 saniyede bir)

- %35 ihtimalle aktif olmayan faction atlanır
- Önce boş bölgelere yayılır (neutral neighbor, %65 ihtimal)
- Aggressive/militarist/expansionist faction'lar oyuncu bölgelerine saldırır (%25 ihtimal, `openDefenseModal` açar)
- Phase 4'te oyuncu %60'tan fazla toprak kontrolündeyse "Great Coalition" uyarısı çıkar (%5 ihtimal)

### 3.4 Alien Attack (Phase 5 — triggerAlienAttack)

ALIEN_UNITS:
- Alien Scout: 300 güç (dalga 1)
- Combat Drone: 600 güç (dalga 2)
- Plasma Warrior: 1200 güç (dalga 2)
- Alien Mothership: 6000 güç (dalga 3)
- Star Crusher: 25000 güç (dalga 4)

Dalga sayısı: `Math.min(4, Math.floor(state.totalTime / 600) + 1)` — her 10 dakikada bir artıyor.  
Oyuncu gücü uzaylı gücünün %50'sinden fazlaysa: saldırı savuşturulur, küçük kayıp.  
Değilse: bölge `rebel` olarak işaretlenir, 60 saniye sonra otomatik geri alınır.

---

## 4. SAVAŞ SİSTEMİ

### 4.1 Güç Hesabı (calcPlayerPower)

1. Tüm birimleri kategoriye göre toplar (land/sea/air/space)
2. Araştırma çarpanları uygular (mechanized +50%, armor_doctrine +60%, vb.)
3. `TERRAIN_MOD` uygular
4. Helicopter synergy: `state.mil.helicopter > 0 && landPow > 0` → landPow ×1.35
5. Active boost `combat_intel` çarpanı eklenir

### 4.2 Savaş Matrisi (COMBAT_MATRIX)

| Saldıran \ Savunan | land | sea | air | space | rebel | alien |
|---------------------|------|-----|-----|-------|-------|-------|
| land | 1.0 | 0.4 | 0.6 | 0.2 | 1.5 | 0.3 |
| sea | 1.4 | 1.0 | 0.5 | 0.2 | 0.6 | 0.4 |
| air | 1.6 | 1.8 | 1.0 | 0.5 | 1.2 | 0.6 |
| space | 2.5 | 2.0 | 2.2 | 1.0 | 1.0 | 2.0 |

### 4.3 Savaş Akışı

1. Oyuncu bölge seçer → "Attack" → `openTroopSelect(regionId)` → kuvvet seçim modalı
2. Terrain preview gösterilir (bonus/ceza/bloke)
3. Onay → `startCombatWithForce(regionId, force)` çağrılır
4. `preForce` snapshot: `Object.keys(force).forEach(uid => { preForce[uid] = state.mil[uid] || 0; })`
5. Auto-round her 5 saniyede bir (combat-howto'da belirtilmiş)
6. "STRIKE!" butonu: bonus hasar her round sonunda uygulanır
7. Savaş bitince: `showCombatSummary(won)` → kayıp listesi gösterilir → buton "✕ Close" olur
8. `retreatOrClose()`: `state.combat.done` ise modalı kapat, değilse geri çekil

### 4.4 Kayıplar

- Global `state.mil` üzerinde takip edilir — bölge bağımsız
- Savaş hasarı direkt `state.mil[uid]` azaltır — kalıcıdır
- `preForce` sadece görüntüleme için tutulur, sıfırlamaz

### 4.5 Defense Modal

Düşman saldırısında `openDefenseModal(targetId, factionId, factionPow)` açılır.  
"Defend!" → `confirmDefense()`, "Retreat" → `retreatDefense()`.

---

## 5. ASKERİYE SİSTEMİ

### 5.1 Birim Kategorileri ve Örnekler (UNIT_DEFS — 36 birim)

**Kara (10 birim):**
- Militia: güç 1, maliyet 15💰+3🌾, limit 200 (başlangıçta mevcut)
- Infantry: güç 5, maliyet 50💰+5⚙️, req: military_org
- Tank: güç 60, maliyet 600💰+30⚙️+15🛢️, req: armor_doctrine (Phase 2)
- Titan Mech: güç 2000, maliyet 50000💰+80⚙️+40💡, limit 3, req: nuclear_weapons+automation+satellite (Phase 5)

**Deniz (9 birim):**
- Patrol Boat: güç 4, req: copper_smelting
- Aircraft Carrier: güç 350, limit 5, req: carrier_ops (Phase 4)
- Ocean Titan: güç 1800, limit 2 (Phase 5)

**Hava (9 birim):**
- Combat Drone: güç 8, req: electronics (Phase 2)
- Strato Bomber: güç 500, limit 5, req: air_doctrine2 (Phase 5)
- Orbital Bomb: güç 1000, limit 3 (Phase 5)

**Uzay (8 birim):**
- Recon Sat: güç 50, req: space_doctrine (Phase 5)
- Star Destroyer: güç 3500, req: galactic_war
- Quantum Weapon: güç 25000, limit 1, req: galactic_war+automation+quantum

### 5.2 Unlock Koşulları

Her birim `unlockPhase` ve `requires[]` (araştırma ID'leri) ile kilitlenir.  
Yalnızca o phase'e girilmişse ve araştırmalar tamamlanmışsa recruit edilebilir.

---

## 6. TEKNOLOJİ AĞACI

### 6.1 Yapı

- 3 dal: `economy`, `science`, `military`
- Toplam: **83 araştırma** (TECH_DEFS dizisi)
- DFS sıralaması `buildTechTreeOrder()` ile — prereq'ler önce
- Her araştırmanın `req:[]` dizisi önceki araştırmaları listeler

### 6.2 Ekonomi Dalı Özeti (27 araştırma)

Tier 0: basic_industry, agriculture, carpentry, copper_smelting  
Tier 1: oil_drilling, logistics, taxation, population_growth, aquaculture, timber_industry  
Tier 2: trade_routes, mercantile_guilds, steel_production, irrigation, urbanization, mining_guilds, energy_grid  
Tier 3: banking, industrial_farming, cold_storage, oil_refining, recycling, rare_earth_extraction  
Tier 4: stock_market, megaprojects, automation, nano_mining  
Tier 5: fusion_reactor, space_mining  
Tier 6: antimatter_economy

### 6.3 Bilim Dalı Özeti (21 araştırma)

Tier 0: writing  
Tier 1: mathematics, medicine, psychology, cryptography  
Tier 2: electronics, chemistry, materials_science, climate_science  
Tier 3: computing, nuclear_theory, genetics, robotics  
Tier 4: nuclear_energy, satellite, ai, cybernetics, fusion_theory  
Tier 5: quantum, bioengineering, nanotech, xenobiology  
Tier 6: warp_theory, singularity

### 6.4 Askeri Dal Özeti (22 araştırma)

Tier 0: military_org, fortification  
Tier 1: psychological_warfare, mechanized, naval_doctrine, guerrilla, strategic_reserves  
Tier 2: military_intelligence, armor_doctrine, air_superiority, amphibious, siege_warfare  
Tier 3: special_forces, land_doctrine2 (Blitzkrieg), naval_domination, stealth_tech, missile_program, biological_warfare  
Tier 4: electronic_warfare, drone_swarms, air_doctrine2, nuclear_weapons, carrier_ops, cyber_warfare  
Tier 5: mech_division, space_doctrine, orbital_defense  
Tier 6: galactic_war, dimensional_tactics

### 6.5 Teknik Not: Emoji İçeren Bloklar

TECH_DEFS, UNIT_DEFS, BUILDING_DEFS, REGIONS dizileri emoji karakter içerir.  
Edit tool ile düzenlemek Unicode mismatch hatası verir.  
**Çözüm:** Python line-range replacement:
```python
with open('world-conquest/index.html', 'r') as f:
    lines = f.readlines()
lines[start:end] = new_content
with open('world-conquest/index.html', 'w') as f:
    f.writelines(lines)
```

---

## 7. YAPI SİSTEMİ (BUILDING_DEFS — 19 yapı)

### 7.1 Erken Oyun (her terrain'de inşa edilebilir)

| Yapı | Maliyet | Efekt | Req Araştırma |
|------|---------|-------|---------------|
| Watchtower | 60💰+10🪵 | +50 savunma; komşuları açar | carpentry |
| Village | 100💰 | +0.5 nüfus/s; +50 pop cap | agriculture |
| Smithy | 120💰+10🪵 | +0.8 demir/s; barracks açar | basic_industry |
| Barracks | 200💰+20⚙️+10🪵 | -20% birim maliyeti; +100 savunma | military_org + smithy |

### 7.2 Terrain Kısıtlı

| Yapı | Terrain | Efekt |
|------|---------|-------|
| Lumber Camp | forest | +1.5 ahşap/s |
| Granary | plains, coast, forest | +500 yiyecek depo; +0.3 yiyecek/s |
| Deep Mine | mountains | +1.5 demir/s; +0.5 bakır/s |
| Naval Base | coast | +30% deniz gücü; naval upkeep -20% |
| Oil Refinery | coast, desert | +2 petrol/s |
| Farm Complex | plains | +3 yiyecek/s; +1 surplus_food/s |

### 7.3 Geç Oyun

Space Base: 5000💰+100💡+30🔮, terrain: tech veya plains, req: space_doctrine → uzay birimleri açılır; rare_earth ×2

---

## 8. RASTGELE OLAYLAR (RANDOM_EVENTS)

Her 90 saniyede %30 ihtimalle tetiklenir. 12 olay:

| Olay | Etki |
|------|------|
| Trade Windfall! | +200 altın |
| Rebel Uprising! | Rastgele bölge rebel olur |
| Oil Discovery! | +80 petrol |
| Famine Warning! | Yiyecek -40 |
| Tech Breakthrough! | Tüm üretim ×1.5, 60 saniye |
| Mercenaries! | +15 infantry bedava |
| Gold Rush! | Altın ×2, 45 saniye |
| Earthquake! | Rastgele bölge savunma -100 |
| Espionage Report! | Sonraki savaş +20%, 180 saniye |
| Peace Offer! | Ceasefire 120 saniye |
| Iron Cache Found! | +100 demir |
| Radiation Leak! | Uranyum üretimi -50%, 60 saniye |

---

## 9. OYUN AŞAMALARI (PHASES)

| Phase | Tetik (bölge oranı) | İsim | Açıklama |
|-------|---------------------|------|---------|
| 1 | 0% (başlangıç) | Foundation | Kuruluş, boş topraklar |
| 2 | %12 (~4 bölge) | Expansion | Rakip uluslarla ilk karşılaşma |
| 3 | %30 (~11 bölge) | Conflict | Büyük savaşlar, ittifaklar |
| 4 | %60 (~21 bölge) | Dominance | Büyük Koalisyon tehdidi |
| 5 | %100 (35/35) | Space Age | Uzaylı istilası başlar |

Phase 5 kazanma koşulu: 35 bölgenin %90'ını (%90 = 31+ bölge) 300 saniye (5 dakika) kesintisiz tutmak.

---

## 10. UI DÜZENİ

```
┌────────────────────────────────────────────────────────────────────┐
│  🌍 World Conquest   [Phase badge]   [💾 Save] [📤 Share] [🔄 Reset] │ ← Header
├────────────────────────────────────────────────────────────────────┤
│  Boost Bar (gizli, boost aktifken 28px görünür)                     │ ← #boost-bar
├────────────────────────────────────────────────────────────────────┤
│  💰 🌾 ⚡ 👥 ⚙️ 🛢️ 💡 ☢️ 🪵 🔧 (+ secondary kaynaklar)          │ ← #res-strip
├───────────────────┬───────────────────────────────┬────────────────┤
│  [Economy]        │  🗺️ World Map                   │ [Region]       │
│  [Military]       │  map-header (bölge adı/istatistik)│ [Military]     │
│  [Science]        │  SVG harita — 35 bölge          │ [Buildings]    │
│                   │  map-footer (tıklama ipucu /   │                │
│  Tech tree        │    "Owned: 0/35")              │ Bölge detay,   │
│  panel            │                               │ birim listesi, │
│  (sol panel)      │                               │ yapılar        │
└───────────────────┴───────────────────────────────┴────────────────┘
```

Sol panel sekmeleri: Economy / Military / Science (tech tree'ler)  
Sağ panel sekmeleri: Region / Military / Buildings

---

## 11. STATE NESNESİ

```js
state = {
  res:    { gold, food, energy, pop, iron, oil, silicon, uranium, wood, copper,
            surplus_food, diamond, rare_earth, ice_core },
  resCap: { /* her kaynak için dinamik cap */ },
  regions: {
    [id]: { owner, troops, neutralDefense, buildings:[], clickPower:1 }
  },
  mil:    { militia, infantry, sniper, apc, artillery, tank, missile_battery, war_robot,
            nuclear_land, titan,
            boat, corvette, frigate, landing_ship, destroyer, submarine, carrier,
            nuclear_sub, ocean_titan,
            drone, fighter, bomber, helicopter, stealth, ew_plane, hypersonic,
            strato_bomber, orbital_bomb,
            recon_sat, defense_sat, space_fighter, laser_platform, star_destroyer,
            asteroid_launcher, nuclear_torpedo, quantum_weapon },
  research:     Set(completedIds),
  buildings:    { [regionId]: [buildingId, ...] },
  phase:        1-5,
  totalTime:    saniye sayacı,
  startedAt:    Date.now(),
  selectedRegion: null,
  combat:       { active, regionId, turns, attackForce, defForce, preForce, done },
  events:       [],
  lastRates:    {},
  activeBoosts: [{ name, type, mult, endsAt }],
  stats:        { clicks, conquered, unitsLost, researchDone, combatsWon },
  firstCombatShown: bool,
}
```

Global değişkenler (state dışında): `alienAttackTimer`, `alienWave`, `_phase5StartTime`, `_winTriggered`

---

## 12. SAVE / LOAD

- **Save key:** `worldconquest_v1`
- **Save içeriği:** res, resCap, regions, mil, research (Array), buildings, phase, totalTime, startedAt, selectedRegion, activeBoosts, stats, firstCombatShown, alienWave, alienAttackTimer, savedAt
- **Tutorial gösterimi:** `localStorage.getItem('worldconquest_v1')` yoksa ilk açılışta tutorial overlay

---

## 13. TEKNIK NOTLAR & BİLİNEN SORUNLAR

### 13.1 renderMapColors() Class Sıfırlama
Bu fonksiyon her çağrıldığında region SVG elemanının `class` attribute'unu sıfırlar. Kıta renk sınıfları (`cont-europe` vb.) her render'da yeniden uygulanmalı. Fragile pattern.

### 13.2 Global state.mil Tasarımı
Askeri birimler bölgeye bağlı değil — tek global havuz. Tüm savaşlarda aynı birimler kullanılıyor. Strateji derinliği sınırlı. Bölge bazlı garnizon sistemi ileride değerlendirilebilir.

### 13.3 Shared-Edge SVG Prensibi
Komşu bölgeler sınır koordinatlarını birebir paylaşır. `paint-order: stroke fill` CSS ile border fill üstünde görünür. `stroke: #111111`, `stroke-width: 1.8px`.

### 13.4 SVG viewBox
`-80 -10 1195 700` — sol kenar negatife uzanıyor (siberia_far_east için: pts `6,18 ... -75,94`).

### 13.5 Region Sayısı Tutarsızlığı
REGIONS dizisi 35 elemanlı. Footer "Owned: 0/35" diyor. Eski dokümanda 38 yazıyordu — güncel kod 35.

---

## 14. GELİŞTİRME DURUMU

### Tamamlanan (MVP)
- [x] SVG dünya haritası (35 bölge, terrain sistemi)
- [x] 14 kaynak (10 primary + 4 secondary üretim)
- [x] Dinamik depo kapları (bölge sayısına göre ölçeklenir)
- [x] 36 askeri birim (kara/deniz/hava/uzay, 4 kategori)
- [x] 83 araştırma (economy/science/military, Tier 0–6)
- [x] 19 bina (terrain kısıtlı + her yerde)
- [x] 7 düşman faction + assignment sistemi
- [x] Enemy AI (expand + player'a saldırı, 45s tick)
- [x] Combat matrix (land/sea/air/space × 6 düşman tipi)
- [x] Terrain modifikatörleri savaşta aktif
- [x] Savaş kayıp takibi (preForce snapshot)
- [x] Savaş özeti + "✕ Close" dönüşümü
- [x] Defense modal (enemy attack)
- [x] 5 game phase + phase banner animasyonu
- [x] Phase 5 alien invasion (4 dalga tipi)
- [x] Kazanma koşulu (35 bölge %90, 5 dk)
- [x] 12 rastgele olay sistemi
- [x] 7 farklı boost tipi
- [x] Tutorial (4 adım)
- [x] Save/Load (localStorage worldconquest_v1)
- [x] Auto-save (30s)
- [x] Share modal
- [x] Reset confirm
- [x] Resource cap bildirim sistemi (60s throttle)
- [x] Desertion (kaynak sıfırlanınca birim kaybı)

### Eksik / Planlanan
- [ ] Unit avantaj matrisi (COMBAT_MATRIX tanımlı ama combat hesabına tam entegre değil)
- [ ] Bölge bazlı garnizon sistemi
- [ ] Diplomatik seçenekler
- [ ] New Game+ modu
- [ ] Gizli olaylar (12 planlanmış, 12 var ama "damage_region" efekti henüz boş)
- [ ] Alien bölge geri alma mekanizması (60s timeout var, ama aktif reconquer yok)

---

## 15. ARAŞTIRMA BULGULARI — HARİTA TASARIMI

1. **Shared-edge prensibi:** Risk, Civilization, EU gibi oyunlarda komşu bölgeler sınır noktalarını paylaşır. SVG polygon'larda aynı koordinatlar kullanılır.
2. **Kıta renk kodlaması:** En yüksek görsel ROI — farklı tonlar sezgisel okuma sağlar.
3. **Grid snap:** Koordinat kaymasını önler.
4. **Label konumları:** `pointer-events:none` ile tıklamayı geçirir.
5. **Stroke sırası:** `paint-order: stroke fill` — sınır fill üstünde görünür.

---

---

## 16. DEĞİŞİKLİK KAYDI

### 2026-06-10 — SVG Tech Tree View (Modal)

**Sol panel:** Değişmedi — `└─` text-indent listesi, sadece unlocked + researched node'lar görünür. "🌳 View Tree" butonu modal açar.

**Modal (openTechTreeModal):**
- 3 branch tab switcher (Economy / Science / Military), tek branch gösterir
- Modal genişliği `92vw / max 1200px` olarak genişletildi
- `computeTreeLayout(techs)` → iterative BFS ile tier (Y ekseni = depth) ve slot (X ekseni = konum) ataması
- `renderTechTreeSVG(techs, containerId, opts)` → SVG 2D grid:
  - Her tier yatayda ortalanır (en geniş tier'e göre)
  - Node'lar: 68×68 px yuvarlak köşeli kare, ikon + kısa isim
  - Bağlantı çizgileri: cubic bezier, parent alt-merkez → child üst-merkez
  - Renk kodları: yeşil (researched) / mavi glow (affordable) / soluk (locked) / sarı (unlocked ama yetersiz kaynak)
  - Glow ring: available ve researched node'larda hafif hale efekti
- Hover tooltip: `ttShowTip()` / `ttHideTip()` — maliyet, açıklama, durum
- **Hover path highlight:** `_ttHighlightPath(techId)` / `_ttResetHighlight()`
  - Ancestor path (gerekli node'lar): **altın** `#f9c74f` bağlantı çizgisi sw=3.5, node'lar parlak
  - Descendant path (bu node'dan açılan node'lar): **mavi** `rgba(61,156,212,0.8)` çizgi sw=2.5, node'lar hafif soluk
  - Hovered node: gold glow (`drop-shadow`)
  - Diğer tüm node ve çizgiler: opacity 0.07–0.12 ile karartılır
  - `_ttAncestors()` ve `_ttDescendants()` recursive walk ile tüm zinciri bulur
  - **Reset:** `_ttHighlightPath` edge'lere dokunmadan önce orijinal `stroke` + `stroke-width` değerlerini `data-orig-stroke` / `data-orig-sw` attribute'larına kaydeder. `_ttResetHighlight` bu kaydedilmiş değerleri geri yükler — base renk sistemi (yeşil/sarı/gri) bozulmaz.
- `researchTech()` → `renderAll()` → `renderLeftPanel()` zinciri (liste güncellemesi)

### 2026-06-10 — Combat Click Effects (STRIKE! Button)

**strikeEnemy() görsel efektleri:**
- **Strike button pulse:** `.striking` CSS sınıfı, `strike-pulse` @keyframes — click'te %88'e scale down + brightness flash, 0.18s
- **Modal red flash:** `.combat-hit-flash` → `combat-flash` @keyframes — modal box'a inset kırmızı box-shadow, 0.22s
- **Floating hit label:** `.strike-float` fixed div, enemy HP bar'ın hemen üzerinde spawn olur; `⚔️ HIT!` / `💥 STRIKE!` / `⚡ BLOW!` rastgele; her 5. click'te `🔥 CRITICAL!` (altın, daha büyük font), 0.9s float-up animasyonu
- **HP bar flicker:** `transition:none` kısa süreli kaldırılır (mevcut davranış korundu)
- Tüm animasyonlar CSS @keyframes tabanlı — DOM cleanup `setTimeout(...remove, 1000)`

*Bu belge her World Conquest geliştirme oturumunda güncellenir.*
