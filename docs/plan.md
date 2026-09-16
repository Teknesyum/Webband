# Webband Overhaul — Çalışma Planı (Fable revize)

Dal: `webband-overhaul`. Kaynak: `srknzl/Webband` main `6d62d5a` (v1.13.1). Danışma: [001](danisma/001-fable-plan-revizyon.md).

Kullanıcı: hepsini yap, dalga onayı yok, sonda denetler. t0 = ben (yönetici). Fable sırası uygulanıyor.

## Kanıtlı ön bulgular
- **#96 kısmen çözülmüş:** `resetMapInteractionState()` zaten var (app.js:5426), `showScreen('map')`
  çağırıyor (app.js:4374). Kalan boşluk okunacak, sadece gerçekten bozuk kısım düzeltilecek.
- **Kuşatma "kale kalıyor" bug'ı ÜRETİLEMEDİ:** siege sırası ground duvar pikseli 12507 → normal
  savaş sonrası `Battle.siege=false`, duvar pikseli 0. Ground doğru yeniden üretiliyor. Şikayet =
  kuşatma sahnesinin çıplak duvar+boş zemin olması. → Dalga 4'te "kuşatma savaşı update" ile zenginleştir.
- 35 issue hâlâ açık; hiçbiri remote'da kapanmamış.

## Dalgalar (Fable sırası)
- **D0 Senkron+triage** ✓ (pull, issue triage, siege repro, forceScore/BALANCE hedefleri).
- **D1 Foundation:** save `version`+`migrate()`; sahne/savaş yaşam döngüsü (`Battle.reset`, #96 tamamla,
  kuşatma bayrağı denetimi, karşılaşma sayısı #116/(5)); tek `Modal` yardımcısı + tımar UI (6/#... çıkış+çift başlık+scroll);
  ESC pause #113; `state.stats` sayaçları (#111/#120/#127 temeli); debug paneli; `window.onerror`+savaş öncesi yedek slot;
  doktor/cerrah koşulu; #116 duyuru=kadro.
- **D2 Denge temeli:** `BALANCE` nesnesi; saf `Game.Force.score(party)`; kademe tablosu (veri);
  "kolay" etiketi güç skoruna; seviye eğrisi #124; headless `Battle.simulate(A,B,n)`; `smoke.html`.
- **D3 Zorluk iskeleti:** `state.settings.difficulty` (5 kademe, dünya+savaş ayrı), `Difficulty.get`, çarpanlar (ayar menüsü).
- **D4 Savaş:** AI rol/öncelik/geri çekilme #109; anti-süvari/itme; ok hızı #118; ~10x zoom + (gerekirse) minimap;
  **kuşatma savaşı update**; ardından (8) afterArmor + (9) asker/çete sayısal denge sim ile; AI zorluk kancası.
- **D5 Harita:** 2x mod → yaya yavaşlatma → çete yoğunluğu #126/#97 → yol olayları #94(+#117) → mareşal/kuşatma dengesi → kenar kaç-prompt.
- **D6 Ekonomi:** her mal al-sat; güçlü asker 3x yem; pahalı yem uzun; katır/eşek yük+sürü debuff; aşırı yük karesel #98; atlı terfi at; pazar UI #103.
- **D7 Field battle:** savaşan lord/mob dursun+destek+ganimet+kılıç ikonu; #119 katılan; #108 kırmızı etiket; #112 kurt sürüsü.
- **D8 Nişan/relic → başarım → boss:** kalıcı efektli nişan (stack yok, esarette kalır, hancı alır, tüm eşyaya açıklama);
  başarımlar #127 (stats'tan); boss revizyonu (map işareti, uniq boss+ödül). Boss zinciri/oyun sonu = ATILDI (ayrı proje).
- **D9a UI cila** (harita odaklı, F11, mobil, kenar menü engellemesin); **9b küçük:** #121 #123 #115 #125 #128 #104 #100-102 #95;
  **9c:** 1v1 modu (#114+#122), perk listesi (skill tree #110 daraltıldı).

## Atılan/ertelenen (Fable)
Drag-drop silüet envanter (mobil düşman → slot'a tıkla-liste), arazi çarpışma (orman=isabet−%X ile geçiştir),
skill tree tam ağaç (→perk listesi), boss zinciri+oyun sonu.

## Kural
Tek yazar/dosya (Edit çakışması yok). app.js+battle.js çekirdeği ben; izole tek dosya işleri (quests.js içerik,
style.css sahne) foundation sonrası ajana. Her dalga sonu tarayıcı doğrulama + headless sim. Perf her dalga karşılaştır.
