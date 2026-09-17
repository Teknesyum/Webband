# 005 — Revizyon Yol Haritası (Danışma: Fable)

Danışan: t0 manager · Model: fable · Tarih: 2026-09-17
Bağlam: sıralı mini-plan 1→4→3→5 bitti (1.19.0); özgün 55-kalemlik plan.md'nin geri kalanı için
roadmap/öncelik/risk danışması.

## Sorulan (girdi, tam metin)

> Webband (build'siz saf-JS tarayıcı RPG, Mount & Blade tarzı, Türkçe arayüz) için büyük bir
> revizyon planının bir bölümünü bitirdim. Senden ROADMAP/ÖNCELİK/RİSK danışmanlığı istiyorum —
> kod değil, karar ve sıralama.
>
> TAMAMLANAN (kullanıcının seçtiği öncelik sırası 1→4→3→5, hepsi push edildi, testler 174/174):
> - Item 1: Skill tree (6 dal, karşıt perkler). — 1.15.0
> - Item 4: Sahne görselleri (köy/kale/şehir silüetleri, han/arena/pazar sahneleri). — 1.17.0
> - Item 3: Boss/relic derinliği — 4 benzersiz boss namla haritada beliriyor, benzersiz satılmaz
>   ödül + kalıcı nişan (relicMod), 4 nişan + 300 nam → Savaş Tanrısı → zafer ekranı. — 1.18.0
> - Item 5: Harita odaklı arayüz — harita tüm ekranı kaplıyor, chrome yarı saydam cam kenar
>   panelleri, mobil + uygulama-içi tam ekran düğmesi. — 1.19.0
>
> HENÜZ AÇIK OLAN BÜYÜK KALEMLER (özgün planın ~55 kaleminden kalanlar, kabaca):
> - Denge/savaş çekirdeği: tek merkezi Game.forceScore; yüksek zırha "1 hasar"; afterArmor taban
>   revizyonu; asker/çete/hasat-çapulcusu statları; asker seviye eğrisi. (duel.js gerçek
>   dengesizlik gösteriyor AMA docs/SYSTEMS.md hedef oranları BAYAT — v0.63.)
> - Savaş AI + arazi: rol/öncelik/geri çekilme AI, anti-süvari, itme-ayrışma, ok hızı, arazi.
> - Dünya/diplomasi: mareşal seferleri çok sık; kuşatmalar uzun; çete eğrisi; yol olayları.
> - Ekonomi: her mal alınıp satılabilsin; yemek gücüne bağlı; katır/eşek yük; atlı terfi at; pazar 1x/5x.
> - Field battle: savaşan lordlara destek, yolda tekli NPC, dolandırıcı satıcı, kırmızı etiket, kurt sürüsü.
> - İçerik/UI: başarım (#127), turnuva→gerçek dövüş (#122), görev+ödül teslim, esc=pause, kule,
>   arena ödülü, savaş sonrası tablo, müzik fade, optimizasyon.
>
> SORULARIM:
> 1. Kalanları hangi SIRAYLA? En yüksek oyuncu-değeri/risk oranıyla ilk 3 dalga.
> 2. Denge çekirdeği en riskli (bayat hedef). Önce SYSTEMS.md hedeflerini ölçümle mi yeniden
>    belirleyelim, yoksa his temelli yeni hedef mi? Somut yol.
> 3. Hangi kalemler kapsam dışı / düşük getirili?
> 4. En büyük 2-3 risk ne, nasıl azaltılır?

## Dönen (çıktı, tam metin)

### 1. Sıralama — üç dalga

**Dalga A — "Dünya nefes alsın" (düşük risk, yüksek his)**
1. Dünya/diplomasi: mareşal sefer sıklığı, kuşatma süresi, çete eğrisi. `sim.js` zaten ölçüyor →
   hedef koy, ayarı çevir, ölç. Tek parametreli değişiklikler.
2. Ekonomi (her mal alınır-satılır, pazar 1x/5x, yük sınırı). `economy.js` var, UI küçük.
3. Field battle küçükler: düşman etiketleri kırmızı, savaşan lordlara destek, yolda katılan NPC.
   Mevcut karşılaşma makinesine ek.

**Dalga B — Savaş çekirdeği (tek büyük riskli kalem, izole)**
4. Önce hedef belirleme adımı (Soru 2). Sonra `afterArmor` + `forceScore` + stat tabanı **tek PR**,
   `duel.js --report` öncesi/sonrası tablosu ile.
5. Asker seviye eğrisi + yemek-güç bağı (forceScore'a bağımlı olduğu için buraya).

**Dalga C — Savaş AI + içerik**
6. Savaş AI: rol/geri çekilme/anti-süvari. Yalnız denge oturduktan sonra — AI değişince duel
   ölçümleri de kayar, ikisini aynı anda oynatma.
7. Esc=pause, savaş sonrası tablo, görev ödül teslimi, arena ödülü, müzik fade — tek tek kapanan
   UI parçaları, aralara serpilir.

Gerekçe: A hızlı sürüm çıkarır ve oyuncuya görünür; B'nin ölçüm altyapısı hazır ama hedefleri yok;
C, B'nin üstüne biner.

### 2. Denge çekirdeğine giriş — somut yol

Bayat hedefi (v0.63) yeniden yazmayı deneme, **yeni hedef tanımı tek ilkeden türesin**:

1. `duel.js --report` çalıştır, mevcut matris `docs/measurements/`e (bugünkü gerçek).
2. Hedefi "kazanma oranı" değil **taş-kağıt-makas kuralı** olarak yaz: aynı kademe eşleşmesi 40–60,
   bir kademe fark 65–80, iki kademe 85+, karşıt tip (mızrak→süvari, ok→hafif) +10. SYSTEMS.md'ye
   6-8 satır, sayı değil aralık.
3. "1 hasar" sorununu tek başına önce çöz: `afterArmor`'a taban (raw'ın %15-20'si) ekle, matrisi
   yeniden çek. Bu tek değişiklik büyük ihtimalle ızgaranın yarısını düzeltir.
4. Kalan ihlalleri stat tablosunda düzelt; her değişiklik sonrası matris diff. `test.js`'e 3-4
   "çapa eşleşme" assertion ekle (ör. Nord Baltacısı vs Kalkanlı 45–60).
5. `sim.js` ile çete/hasat çapulcusu oranını dünya ölçeğinde doğrula (oyuncu 10 askerle 3. gün
   ezilmiyor mu).

Oyun-hissi hedefi ayrıca koyma; aralık kuralı hissi zaten kodlar, sayı tartışmasını bitirir.

### 3. Kapsam dışı / düşük getirili

- **Turnuva→gerçek dövüş (#122):** ayrı engine, TournamentMinigame yeniden. Yüksek maliyet, mevcut
  mini oyun çalışıyor. Ertele.
- **Başarım sistemi 50 madde (#127):** 10-12 ile başla, geri kalanı at. Veri işi, mekanik değil.
- **Birim itme-ayrışma + ok hızı + arazi çarpışmaları:** compositor darboğazlı motorda fizik ekleme
  = perf riski, görünür kazanç az. Yalnız itme-ayrışmayı basit ayrım kuvvetiyle yap, gerisini kes.
- **Kurt sürüsü davranışı, dolandırıcı satıcı, gözetleme kulesi:** birer olay; olay tablosuna satır
  olarak ekle, sistem yazma.
- **Optimizasyon geçişi:** ölçülmüş bir sorun yok (1.2 ms/16.7). Kalemden düş, framegate yeşilse dokunma.
- **Katır/eşek yük sınırı + atlı terfisi için at:** envanter sürtünmesi; oyuncu bunu ceza olarak
  hisseder. Yalnız "at gerekli terfi" kalsın, yük sınırı at.

### 4. En büyük riskler

1. **Denge + AI eşzamanlı değişimi** → hangisi neyi bozdu bilinmez. Azaltma: B ve C ayrı sürüm,
   her biri öncesi/sonrası `duel --report` diff'i commit'te.
2. **Save migrasyon zinciri kırılması** (yeni alanlar: ödül teslim adımı, başarım, mal listesi).
   Azaltma: her yeni alan `ensureX()`; `test.js`'e "1.15 kaydı yüklenir" fixture'ı ekle — şu an
   yoksa en ucuz sigorta bu.
3. **SYSTEMS.md "Measured" satırlarının bayatlaması** (zaten oldu). Azaltma: her sürüm kapanışında
   tek komut `sim + duel + economy --report`, CHANGELOG'a ölçüm dosyası linki zorunlu.
