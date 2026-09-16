# Danışma 001 — Fable, Plan Revizyonu

Tarih: 2026-09-17 · Danışman: Fable (model=fable) · Maliyet: ~59k token, ~143 sn, 0 araç.

## Gönderilen (girdi, tam metin)

Sen bir oyun mimarisi/proje danışmanısın. Sana bir revizyon planı veriyorum, dosya OKUMA —
sadece verdiğim metne dayanarak değerlendir. Türkçe cevap ver.

BAĞLAM: `srknzl/Webband` — build'siz, tek sayfa, vanilya JS tarayıcı RPG'si (Mount & Blade
Warband tarzı). ~10k satır: app.js, battle.js, nobles.js, quests.js. Tek global `state`,
localStorage save. Sahibinden yetkili tek geliştirici çalışıyor. İş: önce bir DEBUG turu,
sonra 35 açık issue (#94–#128) + kullanıcının devasa özellik/denge isteği. Kullanıcı "önce
plan sonra uygulama; işi savsaklama; optimize kal" dedi.

KRİTİK NOT: Yerel klon bayat (shallow, app.js 5253 satır) ama remote main ~9800+ satır. Yani
ilk iş remote'u çekmek (Dalga 0), satır numaralarını yeniden doğrulamak.

[Ardından 0–9 dalga özeti ve 6 soru gönderildi — sorular: (1) dalga sıralaması/bağımlılık,
(2) ertelenebilir/atılabilir kalem, (3) debug turu eksik stabilite düzeltmesi, (4) forceScore
merkezîleştirme zorluktan önce mi, (5) build'siz vanilya tek-state mimari tuzak, (6) atlanan.]

## Dönen (çıktı, tam metin)

### 1. Sıralama / gruplama — düzeltilecek yerler
- Dalga 3 (zorluk) Dalga 4'e (AI) bağımlı, ters sırada. Dalga 3'te sadece iskelet kur
  (`state.settings.difficulty`, `Difficulty.get(key)`, dünya/savaş oran çarpanları). AI kademesi
  Dalga 4'te kanca. Dalga 3 yarım günlük iş olur.
- Dalga 2'deki (8) zırh/hasar ve (9) asker stat dengesi erken. AI/ok/süvari efektif DPS'i
  değiştirir; önce dengelersen iki kez dengelersin. Dalga 2'de sadece kademe tablosunu (veri)
  tanımla; sayıları Dalga 4'ten sonra ayarla.
- (18) envanter sürükle-bırak Dalga 4'e ait değil; UI overhaul'a taşı (ve "at" listesinde).
- Dalga 5 içi sıra ters. Harita 2x (21) her şeyi etkiler. Sıra: (21)→(23) yaya→(19) çete
  yoğunluğu→(20) yol olayları→(24) mareşal/kuşatma→(22) kenar prompt.
- (25) aşırı yük karesel → Dalga 6 (katır/eşek ile aynı sistem, tek seferde).
- #117 dolandırıcı satıcı → Dalga 5 (20) yol olayları.
- #111 esir sayacı, #120 savaş sonrası tablo, #127 başarımlar aynı temele oturur: `state.stats`.
  Sayaç nesnesini Dalga 1'de aç, her dalga artırsın; başarımlar Dalga 8'de ucuz olur.
- #114 düello + #122 turnuva gerçek dövüş = tek alt sistem (1v1 savaş modu). Birlikte, geç.
- UI overhaul en sonda tuzak. Kabuğu (harita odaklı layout, kenar menü, tek `Modal`) Dalga 1'e
  al; Dalga 9 sadece cila. Tımar hatası (6) bu `Modal`'ın ilk müşterisi.
- #113 ESC pause → Dalga 1 (oyun döngüsü/stabilite).

### 2. Ertele / at
- (18) sürükle-bırak silüet envanter: pahalı, mobil düşman. Yerine "silüet üstünde slota tıkla
  → liste aç". Ertele.
- (16) arazi çarpışma/orman oku: veri modeli yoksa büyük. En ucuz: "orman = ok isabeti −%X".
- #110 skill tree: forceScore ve tüm dengeyi açar. "Perk listesi" (5-8 pasif) olarak daralt ya da at.
- Boss "parçalar→bossların bossu→oyun biter": oyun sonu tasarımı, ayrı proje, en sona. Nişan/relic
  ayır ve önce yap.
- #100-102 sahne görselleri: asset işi; paralel ya da ertele.
- (17) savaş minimap: 10x zoom sonrası gerçekten gerekirse.

### 3. Debug turu — eksik kritik kalemler
- Save şeması sürümü + `migrate()`. En büyük atlama. Zorluk/nişan/stats/2x harita eski kayıtları
  kırar. `state.version` + sıralı migrate Dalga 1'de olmalı.
- (3) kuşatma bayrağı, (5) karşılaşma sayısı, #96 kamera aynı kökten: sahne yaşam döngüsü yok.
  Tek `Scene.enter/leave` + `Battle.reset()`; tüm setInterval/rAF temizliğini denetle (sızıntı →
  N savaş sonra yavaşlama).
- Kamera soyutlaması #96'yı çözerken kur: dünya→ekran tek transform (10x zoom + 2x harita + minimap için).
- Debug paneli: altın ver, savaş başlat (A vs B), zorluk seç, tımar aç.
- `window.onerror` + savaş öncesi yedek slot.
- RNG tohumlama (opsiyonel) — #116 gibi hataları tekrar üretmek için.

### 4. forceScore → zorluk sırası
Doğru. İki şart: (a) `forceScore(party)` saf fonksiyon, `state` okumasın; zorluk çarpanı çağıran
tarafta uygulansın. (b) Tüm sayısal sabitler `balance.js` içinde tek `BALANCE` nesnesine; zorluk =
`BALANCE` üstüne çarpan katmanı.

### 5. Mimari uyarılar (build'siz vanilya + tek state)
- Yeni global açma; `Game.Force`, `Game.Difficulty`, `Game.Economy` gibi IIFE namespace; script
  sırası sabit, yükleme anında çapraz çağrı yok.
- Headless savaş simülatörü: `battle.js`'te sim adımını render'dan ayır; `simulate(A,B,n)` kazanma
  oranı bassın. En yüksek kaldıraçlı araç.
- `smoke.html`: scriptleri yükleyip saf fonksiyonları (forceScore, afterArmor, seviye eğrisi,
  migrate) assert eden tek sayfa. Ucuz, her dalgada koş.
- localStorage 5 MB: türetilmiş veriyi save'e yazma; savaş durumu `state` dışında.
- Mobil: `dvh`, touch; drag-drop yok.
- Dalga başına branch + PR, "fixes #N". 35 issue tek PR'da kapanmasın.
- Perf taban ölçümü Dalga 0'da (fps, tick, save boyutu); her dalga sonunda karşılaştır.

### 6. Bariz atlananlar
- Dalga 0'a: 35 issue'nun hangileri remote main'de zaten kapanmış/mükerrer? (yerel bayattı).
- Dalga başına manuel regresyon senaryosu (yeni oyun→ilk çatışma→kuşatma→tımar→kayıt/yükle).
- Dalga 9'u böl: 9a UI kabuk cilası, 9b küçük özellikler (#121 #123 #115 #125 #128 #104 #105-107),
  9c 1v1 modu + skill/perk.
- Dalga başına kaba ağırlık: D0 küçük, D1 orta, D2 küçük-orta, D3 küçük, D4 büyük, D5 orta, D6 orta,
  D7 orta, D8 büyük (boss zinciri atılırsa orta), D9 büyük.

### Önerilen revize sıra
0. Senkron + issue triage + perf/save taban ölçümü
1. Yaşam döngüsü (sahne/savaş reset, kamera soyutlaması, #96, (3), (5)), `migrate()`, `Modal` +
   UI kabuk, ESC pause, `state.stats`, debug paneli, onerror + yedek slot, doktor koşulu, #116
2. `BALANCE` + saf `forceScore` + kademe tablosu (veri) + "kolay" etiketi + seviye eğrisi +
   headless sim + smoke.html
3. Zorluk iskeleti (ayar, save, dünya/savaş çarpanları)
4. Savaş AI + anti-süvari + ok hızı + 10x zoom → ardından (8)(9) sayısal denge sim ile → AI zorluk kancası
5. Harita 2x → yaya hızı → çete yoğunluğu → yol olayları (+#117) → mareşal/kuşatma → kenar prompt
6. Ekonomi + yük/katır/aşırı yük + pazar UI
7. Field battle (#119 #108 #112, lord desteği, ganimet)
8. Nişan/relic → başarımlar (stats'tan) → boss (zincir olmadan, tek boss + ödül)
9a/9b/9c yukarıdaki gibi; atılanlar: drag-drop envanter, arazi çarpışma, skill tree, boss zinciri/oyun sonu.
