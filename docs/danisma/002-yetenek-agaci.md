# Danışma 002 — Yetenek Ağacı (#110) Tasarımı

Model: fable (Agent aracı). Tarih: 2026-09-17. Süre ~81 sn, ~62.4k token.

## Girdi (aynen)

> Webband adlı Mount&Blade: Warband tarzı tek-sayfa tarayıcı RPG için bir YETENEK AĞACI (skill tree, #110) TASARLA. Kod yazma — sadece tam bir tasarım verisi üret. Dosya okuman yok; gereken tüm olgular aşağıda. Cevabın TÜRKÇE olsun. Somut ol, JSON benzeri tablo ver.
>
> ## Mevcut sistemler (gerçek kod)
> - Nitelikler (5): str(Güç), agi(Çeviklik), int(Zekâ), cha(Liderlik), vit(Dirayet). Hedef/efektif mekaniği.
> - Yeterlilikler (15) ve GERÇEK etkileri:
>   - oneHanded: kılıç hasarı ×(0.35..0.75)
>   - twoHanded: çift el hasarı ×(0.35..0.75)
>   - polearm: mızrak hasarı ×(0.35..0.75), atlı şarjda ×2.6
>   - bow: ok hasarı ×(0.5..1.0), savaş başına 24+lvl*2 ok
>   - riding: atlı savaş hızı
>   - athletics: yaya savaş hızı
>   - leadership: grup kapasitesi +4/lvl, moral +3/lvl
>   - persuasion: diyalog/drahoma
>   - surgery: ölen askerin yaralı kurtulma şansı %35..75
>   - prisonerMgmt: esir kapasitesi, kaçış şansı
>   - pathfinding: harita hızı +%2/lvl
>   - spotting: görüş +25/lvl
>   - trade: alış indirim/satış prim %2/lvl (max %25)
>   - looting: savaş ganimeti +%4/lvl
>   - trainer: her gün N askere +1 XP
> - Seviye ekonomisi: her seviye +1 nitelik puanı, +3 odak puanı (odak yeterlilik öğrenme hızına gider, 0-5).
> - Ek gerçek sistemler: maxHp (+10/seviye), HP regen (Dirayet'e bağlı), maaş borcu (moral cezası), yemek tüketimi (güçlü asker çok yer), aşırı yük (hız cezası), atlı asker terfisi için at, nam (renown) grup kapasitesine katkı.
>
> ## İSTENEN YAPI
> - 6 dal, her dal 5 kademe, her kademe 2 karşıt perk (oyuncu birini seçer) = dal başına 10 perk = TOPLAM 60 perk.
> - Karşıt perkler gerçek bir ödünleşim (trade-off) sunmalı.
> - Her perk etkisini sabit modifiye sözlüğünden BİR (bazen iki) anahtara sayısal katkı olarak ver:
>   dmg1h, dmg2h, dmgPolearm, dmgBow, arrowCount, meleeSpeed, footSpeed, ridingSpeed, chargeDmg,
>   partyCap, moraleBonus, prisonerCap, escapeReduce, healChance, mapSpeed, vision, tradeEdge,
>   loot, trainXp, foodUse, maxHpBonus, hpRegen, wageReduce, renownGain, blockAngle.
> - 6 dal önerisi + perk puanı ekonomisi + çıktı formatı `id | İsim | açıklama | {mod}`.

## Çıktı (aynen)

# Webband Yetenek Ağacı (#110) — Tasarım Verisi

Format: `id | İsim | Açıklama | {mod}`. Her kademede A ve B karşıttır; biri seçilir.

## 1. Kılıç Ustalığı (`melee`)

**K1**
`melee_edge_a | Keskin Kenar | Tek el silah hasarı artar. | {dmg1h: 0.10}`
`melee_guard_b | Sağlam Duruş | Savuşturma açısı genişler. | {blockAngle: 15}`

**K2**
`melee_heavy_a | Ağır Darbe | Çift el silah hasarı artar. | {dmg2h: 0.12}`
`melee_swift_b | Hızlı Bilek | Yakın dövüş vuruş hızı artar. | {meleeSpeed: 0.08}`

**K3**
`melee_spear_a | Mızrak Dikişi | Mızrak hasarı artar. | {dmgPolearm: 0.12}`
`melee_parry_b | Savuşturucu | Savuşturma açısı daha da genişler. | {blockAngle: 20}`

**K4**
`melee_fury_a | Öfke | Tek ve çift el hasarı birlikte artar. | {dmg1h: 0.10, dmg2h: 0.10}`
`melee_ironskin_b | Demir Ten | Azami can artar. | {maxHpBonus: 20}`

**K5**
`melee_master_a | Silah Ustası | Üç yakın dövüş silahı da güçlenir. | {dmg1h: 0.15, dmg2h: 0.15, dmgPolearm: 0.15}`
`melee_bulwark_b | Kale | Savuşturma ve can birlikte artar. | {blockAngle: 30, maxHpBonus: 15}`

## 2. Süvari & Okçu (`ranger`)

**K1**
`ranger_quiver_a | Geniş Sadak | Savaş başına ok sayısı artar. | {arrowCount: 8}`
`ranger_saddle_b | Sağlam Eyer | Atlı savaş hızı artar. | {ridingSpeed: 0.08}`

**K2**
`ranger_aim_a | Keskin Nişan | Ok hasarı artar. | {dmgBow: 0.10}`
`ranger_lance_b | Mızrak Şarjı | Atlı şarj hasarı artar. | {chargeDmg: 0.25}`

**K3**
`ranger_fleet_a | Hafif Ayak | Yaya savaş hızı artar. | {footSpeed: 0.10}`
`ranger_horseman_b | Doğuştan Binici | Atlı savaş hızı daha da artar. | {ridingSpeed: 0.12}`

**K4**
`ranger_volley_a | Yaylım | Ok hasarı ve ok sayısı birlikte artar. | {dmgBow: 0.10, arrowCount: 6}`
`ranger_shock_b | Şok Hücumu | Atlı şarj hasarı büyük ölçüde artar. | {chargeDmg: 0.40}`

**K5**
`ranger_eagle_a | Kartal Göz | Okçuluğun zirvesi: hasar ve sadak. | {dmgBow: 0.20, arrowCount: 10}`
`ranger_stormrider_b | Fırtına Binicisi | Süvariliğin zirvesi: hız ve şarj. | {ridingSpeed: 0.15, chargeDmg: 0.30}`

## 3. Komuta (`cmd`)

**K1**
`cmd_banner_a | Sancak | Grup kapasitesi artar. | {partyCap: 6}`
`cmd_thrift_b | Tutumlu Kâhya | Maaş gideri azalır. | {wageReduce: 8}`

**K2**
`cmd_spirit_a | Ocak Başı | Grup morali artar. | {moraleBonus: 6}`
`cmd_drill_b | Talim | Her gün daha çok asker XP alır. | {trainXp: 3}`

**K3**
`cmd_host_a | Kalabalık Ordu | Grup kapasitesi daha da artar. | {partyCap: 10}`
`cmd_fame_b | Nam Salan | Nam kazanımı artar. | {renownGain: 15}`

**K4**
`cmd_veterans_a | Gaziler | Eğitim ve moral birlikte artar. | {trainXp: 5, moraleBonus: 3}`
`cmd_paymaster_b | Hazinedar | Maaş gideri daha da azalır. | {wageReduce: 12}`

**K5**
`cmd_warlord_a | Savaş Beyi | Büyük ordu, yüksek moral. | {partyCap: 15, moraleBonus: 5}`
`cmd_legend_b | Efsane | Nam ve tasarruf birlikte. | {renownGain: 25, wageReduce: 8}`

## 4. Sıhhiye & Zindan (`med`)

**K1**
`med_bandage_a | Sargı | Ölen askerin yaralı kurtulma şansı artar. | {healChance: 8}`
`med_tough_b | Sert Deri | Azami can artar. | {maxHpBonus: 15}`

**K2**
`med_rest_a | Dinlenme | Can yenilenme hızı artar. | {hpRegen: 20}`
`med_chains_b | Zincirler | Esir kapasitesi artar. | {prisonerCap: 6}`

**K3**
`med_surgeon_a | Alan Cerrahı | Kurtulma şansı daha da artar. | {healChance: 10}`
`med_warden_b | Gardiyan | Esir kaçış şansı azalır. | {escapeReduce: 20}`

**K4**
`med_vigor_a | Kuvvet | Can ve yenilenme birlikte artar. | {maxHpBonus: 25, hpRegen: 10}`
`med_dungeon_b | Zindan | Esir kapasitesi ve tutma birlikte artar. | {prisonerCap: 10, escapeReduce: 15}`

**K5**
`med_miracle_a | Mucize | Sıhhiyenin zirvesi. | {healChance: 15, hpRegen: 25}`
`med_slaver_b | Esirci | Zindancılığın zirvesi. | {prisonerCap: 15, escapeReduce: 25}`

## 5. İz Sürme (`scout`)

**K1**
`scout_stride_a | Uzun Adım | Harita hızı artar. | {mapSpeed: 5}`
`scout_eye_b | Keskin Bakış | Görüş menzili artar. | {vision: 40}`

**K2**
`scout_ration_a | Kısa Kumanya | Yemek tüketimi azalır. | {foodUse: 0.90}`
`scout_runner_b | Koşucu | Yaya savaş hızı artar. | {footSpeed: 0.08}`

**K3**
`scout_trail_a | Patika Bilgisi | Harita hızı daha da artar. | {mapSpeed: 8}`
`scout_watch_b | Nöbetçi | Görüş menzili daha da artar. | {vision: 60}`

**K4**
`scout_forage_a | Toplayıcı | Yemek tüketimi belirgin azalır. | {foodUse: 0.85}`
`scout_pace_b | Zorlu Yürüyüş | Harita ve yaya hızı birlikte artar. | {mapSpeed: 6, footSpeed: 0.06}`

**K5**
`scout_ghost_a | Hayalet | Hızlı ve az yiyen grup. | {mapSpeed: 12, foodUse: 0.90}`
`scout_hawk_b | Şahin | Uzağı gören, hızlı savaşan. | {vision: 100, footSpeed: 0.08}`

## 6. Çapul & Ticaret (`raid`)

**K1**
`raid_pillage_a | Yağma | Savaş ganimeti artar. | {loot: 8}`
`raid_haggle_b | Pazarlık | Alış-satış marjı artar. | {tradeEdge: 4}`

**K2**
`raid_stockpile_a | Ambar | Yemek tüketimi azalır. | {foodUse: 0.92}`
`raid_bookkeeper_b | Defterdar | Alış-satış marjı daha da artar. | {tradeEdge: 5}`

**K3**
`raid_plunder_a | Talan | Ganimet belirgin artar. | {loot: 12}`
`raid_caravan_b | Kervan Ağı | Marj daha da artar. | {tradeEdge: 6}`

**K4**
`raid_notorious_a | Adı Çıkmış | Ganimet ve nam birlikte artar. | {loot: 8, renownGain: 10}`
`raid_merchant_b | Tüccar Prens | Marj ve nam birlikte artar. | {tradeEdge: 6, renownGain: 10}`

**K5**
`raid_chieftain_a | Çapulcu Reisi | Çapulun zirvesi. | {loot: 15, renownGain: 10}`
`raid_magnate_b | Ticaret Baronu | Ticaretin zirvesi. | {tradeEdge: 8, foodUse: 0.90}`

---

## Perk Puanı Kuralı

- `perkPoints`: 2. seviyeden başlayarak her 2 seviyede 1 puan (`floor(level / 2)`). Maks seviye 40 → 20 puan = 60 perk'in üçte biri, tam 4 dal.
- Kademe kilidi: EVET. Bir dalda K(n) için aynı dalda K(n-1)'den A veya B alınmış olmalı.
- Perk toplamları yeterlilik çarpanına toplanarak eklenir; foodUse çarpılarak yığılır.

## Denge Notu

Saf saldırı hattı (melee A×5) tek el hasarına toplam +0.35 verir; yeterliliğin kendi aralığının yarısı. tradeEdge tüm B'lerle +29; toplam tavan %40'a çıkar. foodUse üç daldan yığılır, taban 0.60. partyCap +31.

---

## t0 kararı (uygulama)

Kullanıcı mid-turn ek kısıt verdi: "skiller birbiriyle ilişkili olsun, zekâ +1 odak versin, bazı seviyeler için min stat/min level olsun, tek stat spam edilmesin". Fable'a tekrar sorulmadı (somut). Uygulama:
- Her perk `req`: dal içi kademe kilidi + min ilgili nitelik + min ilgili yeterlilik seviyesi.
- Zekâ (int): efektif her puan +1 odak puanı verir (spam kırıcı; tek dövüş statı yerine INT ödüllenir).
- Perk puanı: `floor(level/2)`.
