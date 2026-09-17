# 003 — Boss ve Relic/Nişan Sistemi Tasarımı

Danışman: Fable (model=fable, Agent aracı). Tarih: 2026-09-17.
Maliyet: subagent_tokens 60.743, süre ~76 sn, tool_uses 0.
Bağlam: yol planı iş 3/4 (boss/relic derinliği, issue #37 relic + #38 boss).

---

## Gönderilen brief (tam metin)

Sen bir oyun tasarım danışmanısın. Mount & Blade: Warband tarzı, saf vanilla-JS, build'siz bir tarayıcı RPG'si (WebBand) için "boss ve relic/nişan" sistemini tasarlamanı istiyorum. Türkçe cevap ver. Kod yazma — tasarım kararı ver; ben uygulayacağım. Somut, SINIRLI ve TEK sürümde bitirilebilir bir tasarım istiyorum (scope patlatma).

### Mevcut altyapı (bunların üstüne kur)
- Eşya modeli: `ITEMS` sözlüğü. Her eşya `{id,name,type,basePrice,icon,...}`. type: food/trade/weapon/shield/armor/helmet/gloves/boots/horse/special. Silah eşyası `{weaponType:'oneHanded|twoHanded|polearm|bow', dmgType:'cut|pierce|blunt', attack:N}`, zırh/kalkan `{defense:N}`, at `{type:'horse'}`. Eşyaların ŞU AN açıklaması (desc) YOK.
- Envanter: `state.player.inventory` = `[{...item, qty}]`. Kuşanılan: `state.player.equipment.{weapon,shield,armor,helmet,gloves,boots,horse}`.
- Mevcut boss sistemi ÇOK ZAYIF: tek bir tekrarlanan "Savaş Tanrısı" bossu var. `boss_map` (5000 dinar, satın alınabilir, 🗺️) eşyası kullanılınca boss dövüşü açıyor; `state.bossEntries` 1-4 arası, her girişte boss seviyesi +5 artıyor. Ödül: para+xp + `lvl51_token` ("Savaş Tanrısı Nişanı", bir askeri Lvl 51'e çıkarır). Boss haritası için 300 nam (renown) kapısı var.
- Savaş motoru `Battle.start(enemyName, enemyCount, bossLevel, faction, siegePlan, auto, enemyBand)`. bossLevel verilince tek dev birim + korumalar spawn ediyor. Boss dövüşünde esir alınmıyor.
- "perkMod(name)" toplayıcı desen: sözlükteki aktif modifikatörleri toplayıp savaş/harita/ekonomi kancalarına besliyor. Relic efektleri de AYNI `relicMod(name)` toplayıcısıyla, sabit modifikatör kelime dağarcığıyla uygulanabilir.
- Harita "site" sistemi: `state.sites`, `SITE_KINDS`. Bosslar harita işaretçisi olabilir.

### İstenenler
Relic (#37): kalıcı efektli çok çeşit; her çeşitten 1 (stack yok); esarette kaybolmaz; sadece Tüccar Mink satın alınabilir, gerisi bosslardan; fazlası hancıya; tüm eşyalara açıklama.
Boss (#38): haritada işaretli kocaman benzersiz bosslar; her boss benzersiz ödül (güçlü ama bozmayan silah/at + relic); parçaları birleştir → bossların bossu → kesince oyun biter; boss erişimi en pahalı.

### İstenen çıktı (net, sayılı)
1. Kaç benzersiz boss, her biri için ad/erişim/güç/benzersiz eşya/relic. 2. Relic kelime dağarcığı (id/ad/açıklama/efekt, sadece relicMod türünden). 3. Bossların bossu mekaniği. 4. Denge tavanı. 5. Mevcut lvl51_token ve tek boss ne olsun. 6. Tek sürüm MVP sınırı.

---

## Fable yanıtı (tam metin)

### 1. Bosslar — 4 benzersiz + 1 final

Erişim tek mekanizma: hepsi `SITE_KINDS`'a `bossLair` olarak girer, **nam eşiğinde haritada belirir** (satın alma yok; harita parasını finale saklıyoruz). Güç `bossLevel` ölçeğinde, `B` = bugünkü 1. giriş seviyesi.

| # | Boss | Nam kapısı | Güç | Benzersiz eşya | Relic |
|---|---|---|---|---|---|
| 1 | **Kurt Ana** (orman ini) | 60 | B | `kurt_disi_hancer` — oneHanded, cut, atk **17**, basePrice 4000 | `kurt_kani` |
| 2 | **Bozkır Hanı** (atlı, step) | 130 | B+5 | `han_kisragi` — horse, mevcut savaş atının **+15%** hız/def'i | `bozkir_tugu` |
| 3 | **Demirci Dev** (dağ ocağı) | 200 | B+10 | `dev_orsu_zirhi` — armor, def **44** (plate 38+6), ağır: mapSpeed -5% | `demir_yurek` |
| 4 | **Korsan Kral** (kıyı kalesi) | 280 | B+15 | `firtina_yayi` — bow, pierce, atk **22** (mevcut en iyi yay ×1.2 tavanı) | `firtina_tilsimi` |
| F | **Savaş Tanrısı** (bossların bossu) | 300 + 4 relic + `boss_map` | B+25, koruma ×2 | yok — **zafer** | yok |

Her boss tek sefer kesilir (`state.bossKills[id]=true`, site işaretçisi "yıkık" olur). Eşyalar `type` mevcut türlerden, sadece `unique:true` bayrağı; hancı satın almaz (`unsellable:true`).

### 2. Relic sözlüğü — 5 adet, `state.player.relics = {id:true}` (envanter dışı → esarette kaybolmaz)

| id | Ad | Açıklama | relicMod |
|---|---|---|---|
| `kurt_kani` | Kurt Kanı | "Sürü senin adımına ayak uydurur." | `mapSpeed +15%` |
| `bozkir_tugu` | Bozkır Tuğu | "Bozkır kanı damarlarda; ordu yılmaz." | `moral +10` |
| `demir_yurek` | Demir Yürek | "Ocakta dövülmüş bir yürek kolay durmaz." | `maxHp +20%` |
| `firtina_tilsimi` | Fırtına Tılsımı | "Her yağma bir fırtına sonrası gibi." | `loot +25%` |
| `tuccar_mink` | Tüccar Mink | "Her pazarda bir dost, her dostta bir indirim." | `tradeEdge +10%` — Hancı/tüccarda **6000** dinar |

Kural: relic düşünce zaten varsa → envantere `type:'special'`, basePrice 1500 kopya (hancıya satılır). Her relic tek modifikatör; kancalar: harita hızı, moral hesabı, hp init, yağma hesabı, alış-satış marjı — 5 satır.

### 3. Bossların bossu

- **Açılış:** `boss_map` kullanımında `Object.keys(relics).filter(bossRelic).length === 4` şartı. Değilse mesaj: "Nişanlar eksik (2/4)". Craft yok, otomatik.
- `boss_map` fiyatı **5000 → 15000**, oyunun en pahalı eşyası; nam 300 kalır.
- **Tekrar yok:** `bossEntries` kaldırılır.
- **Zafer:** `Battle` win callback'inde `state.victory=true`, `state.victoryDay=day`; mevcut oyun-sonu/ölüm ekranı şablonunu kopyalayıp "Zafer" başlığı + gün/nam/altın/kesilen boss listesi + "Devam Et" düğmesi. Save'e `victory` yazılır, başlıkta ⚔ rozeti. En ucuz yol bu; yeni akış yok.

### 4. Denge tavanları

- Silah: en iyi mevcut ×**1.2** (20 → 24 üst sınır; 17 ve 22 verdim, yay içerik açığı yüzünden tavana daha yakın).
- Zırh: plate **+6** (44), bedeli mapSpeed -5% — "güçlü ama düşündüren".
- At: **+15%**, üstü yok.
- Relic yüzdeleri 10-25 bandında; toplam 4 relic ile oyuncu ~%20 daha hızlı/dayanıklı, sayısal hasar +0. Final boss B+25 bunu karşılar.

### 5. `lvl51_token` ve eski boss

- Savaş Tanrısı **korunur**, finale terfi eder (isim/ikon hazır, sıfır yeni asset).
- `lvl51_token` (Savaş Tanrısı Nişanı) **relic değil, tüketilebilir kalır**; 4 boss'un her biri 1 tane düşürür (toplam 4 = 4 elit asker). Final de vermez. Açıklama: "Bir askeri 51. seviyeye yükseltir. Tek kullanımlık."
- `bossEntries` ve +5 döngüsü silinir.

### 6. Tek sürüm MVP sınırı

**Sürüm 1 (yeter):** 4 bossLair site + nam gate, 4 unique eşya, 5 relic + `relicMod`, `state.player.relics`, boss_map şartı+fiyat, zafer ekranı, **tüm ITEMS'a `desc`** (mevcut ~40 eşya için tek satırlık metin), envanter tooltip'inde desc gösterimi.

**Faz 2:** boss'a özel koruma birlikleri, boss can çubuğu/mesajları, relic vitrini ekranı, boss sitelerinde yeniden doğan alt bosslar, 6-7. relic (`wageReduce`, `prisonerCap`).

Maliyet tahmini: ~350 satır JS + ~50 satır desc verisi, tek oturum.

---

## t0 kararı (uygulama notu)

Fable'ın MVP sınırını (Sürüm 1) aynen uyguluyorum. Sapmalar:
- Erişim: Fable "nam eşiğinde haritada belirir, satın alma yok" diyor. Mevcut `boss_map` satın-alma akışını finale saklıyoruz (Fable de öyle diyor); 4 boss site olarak nam kapısıyla haritada belirir.
- `relicMod(name)` toplayıcısı `perkMod` ile birebir aynı desende; efektler: mapSpeed, moralBonus, maxHpBonus(%'yi çarpan olarak), loot, tradeEdge — hepsi zaten var olan kancalara eklenir.
- Eski `lvl51_token` korunur; 4 bossun her biri 1 düşürür; `bossEntries`/+5 döngüsü ve tek-tekrarlı boss kaldırılır.
- Zafer: mevcut ölüm/oyun-sonu ekranı şablonu kopyalanır.
