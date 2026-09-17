# 004 — Harita Odaklı Arayüz Revizyonu (Danışma: Fable)

Danışan: t0 manager · Model: fable · Tarih: 2026-09-17
Plan kalemi: item 5 (Harita UI overhaul, [K] — GitHub #40 değil).

## Sorulan (girdi, tam metin)

> Webband adlı, build'siz saf-JS tarayıcı RPG'sinin (Mount & Blade tarzı, Türkçe arayüz)
> harita ekranı arayüzünü elden geçireceğim. Senden MVP bir tasarım kararı istiyorum — en
> ucuz, işe yarar sürüm. Kod yazma, tasarım/karar ver.
>
> HEDEF (kullanıcının isteği): "Harita odaklı arayüz. Kenar menüleri ve çerçeveler haritayı
> kapatmasın/engellemesin. Mobilde de çalışsın. F11 tam ekran öner."
>
> MEVCUT DURUM (kod denetimiyle saptandı, olgular):
> - Ekran dikey flex kolon: üstte `#top-bar` (zaman/para/yemek/nam/can/grup/yük/moral/seviye/hız
>   gösteren HUD çipleri satırı), altında yatay `.content-area`.
> - `.content-area` içinde solda `#sidebar` (160px, dikey nav: Harita/Karakter/Grup/Envanter/
>   Görevler + Kayıtlar/Ses/Ayarlar + "⋯ Daha"), sağda `#view-container` → `#map-canvas`.
> - Kritik: top-bar ve sidebar **normal akışta**, haritanın üstüne binmiyorlar; harita canvas'ı
>   yalnızca **artan dikdörtgene** göre boyutlanıyor (parent clientWidth/Height). Yani menüler
>   haritayı "örtmüyor" ama **haritadan yer çalıyor** — harita hiçbir zaman tüm ekranı doldurmuyor.
> - Haritanın üstüne binen tek şeyler: `#map-hud` (sol-alt köşe, arazi+kadro rozeti, tıklama
>   geçirgen) ve imleç ipucu `#map-tooltip` — ikisi de pointer-events:none.
> - Mobil (max-width 820px): sidebar alt şeride iniyor (yatay kaydırmalı), çipler küçülüyor. Ama
>   harita için "tam ekran" yok — üst bar + alt nav dikeyde hâlâ yer yiyor. Savaşta `body.in-battle`
>   ile chrome gizleniyor (savaşa özel), haritada böyle bir şey yok.
> - Uygulama-içi tam ekran YOK; yalnız native F11 için bir ipucu balonu var (`#f11-hint`).
> - Cam-panel ("glass-panel") teması ve CSS değişkenleri mevcut. Dokunmatik hedef min 44px kuralı
>   var. `resizeCanvases()` canvas'ı parent boyutundan okuyor — layout değişince bu varsayım korunmalı.
>
> SORULARIM (MVP kapsamı belirle):
> 1. Harita tüm görünümü doldursun, chrome haritanın ÜSTÜNE yarı saydam cam kenar panelleri olarak
>    mı yüzsün? Yoksa mevcut "yer çalan" düzen kalsın, sadece daralt+gizle-aç düğmesi mi eklensin?
> 2. Yüzen panel yaklaşımı seçilirse: üst bar ve nav nerede dursun? Otomatik solup elle açılan mı,
>    hep görünür yarı saydam mı? Haritanın ortası her zaman açık kalmalı.
> 3. Mobil: masaüstüyle aynı yüzen düzen mi, yoksa savaştaki gibi bir "harita tam ekran" toggle'ı mı?
> 4. Uygulama-içi tam ekran düğmesi (requestFullscreen) ekleyeyim mi, nereye? F11 ipucuyla ilişkisi?
> 5. Bu MVP'de KAPSAM DIŞI bırakılması gerekenler neler?

## Dönen (çıktı, tam metin)

**1. Yüzen cam paneller mi, daralt/gizle mi?**
Karar: **Yüzen paneller (harita tam viewport, chrome üstte overlay).** Gerekçe: "daralt+gizle" iki
durumu da (açık/kapalı) ayrı ayrı doğru göstermeyi ve resize'ı iki kez ele almayı gerektirir;
overlay ise tek CSS değişikliği (`#view-container` → `position:fixed; inset:0`, chrome
`position:fixed`) ve `resizeCanvases()` parent-boyut varsayımı olduğu gibi çalışır.

**2. Konum ve görünürlük**
Karar: **Üst bar üst kenarda, nav sol kenarda dikey ikon şeridi (48px), ikisi de hep görünür ama
yarı saydam (`glass-panel`, hover/dokunmada tam opaklık).** Gerekçe: Otomatik solma "nerede menü?"
sorununu ve JS zamanlayıcı yükünü getirir; sabit yarı saydam kenarlar sıfır JS'le merkezi her zaman
açık bırakır. Sidebar 160px → 48px ikon şeridi, etiketler `title`/tooltip ile; `#map-hud` sol-alt
köşede kalır ama nav şeridinin sağına kaydırılır (`left:56px`).

**3. Mobil**
Karar: **Aynı yüzen düzen; nav alt kenara yatay ikon şeridi (mevcut alt-şerit davranışı korunur),
top-bar üstte tek satır, taşan çipler yatay kaydırır.** Gerekçe: Savaştaki gibi ayrı bir toggle
ikinci durum makinesi demek; overlay zaten haritayı tam ekran yaptığı için toggle'a gerek kalmaz.
Alt şerit min 44px, `env(safe-area-inset-bottom)` padding'i.

**4. Uygulama-içi tam ekran düğmesi**
Karar: **Evet, tek düğme (⛶) nav şeridinin en altına; `document.fullscreenElement` varsa
`exitFullscreen`, yoksa `documentElement.requestFullscreen()`.** Gerekçe: F11 mobilde yok, düğme ise
her yerde çalışır. Mevcut `#f11-hint` balonu kalır ama yalnız masaüstünde ilk açılışta gösterilir;
düğme tıklanınca balon bir daha çıkmaz (`localStorage` bayrağı zaten varsa kullan).

**5. Kapsam dışı**
- Panel sürükleme/yeniden konumlandırma, kullanıcı tarafından özelleştirilebilir düzen.
- Otomatik solma/idle-timer, "chrome'u tamamen gizle" kısayolu.
- Çip içeriği/HUD tasarımı değişikliği (yalnız yerleşim değişir).
- Karakter/Envanter vb. diğer görünümlerde düzen değişikliği — overlay yalnız harita görünümü
  aktifken (`body.view-map` sınıfı) uygulanır; diğer görünümler eski akış düzeninde kalır.
- Savaş ekranı (`in-battle`) dokunulmaz.
- Landscape/portrait özel kırılımlar; tek breakpoint (820px) kalır.

## Sürüm 1 (uygulanacak MVP)

1. `body.view-map` sınıfı: harita görünümüne geçince ekle, çıkınca kaldır (view-switch fonksiyonunda).
2. CSS (`body.view-map` altında):
   - `#view-container { position:fixed; inset:0; }` — sınıf değişince bir kez `resizeCanvases()`.
   - `#top-bar { position:fixed; top:0; left:0; right:0; z-index:10; }` + glass, `opacity:.85`, `:hover{opacity:1}`.
   - `#sidebar { position:fixed; top:var(--topbar-h); bottom:0; left:0; width:48px; z-index:10; }` ikon şeridi.
   - `#map-hud { left:56px }`.
   - `@media (max-width:820px)`: sidebar alt kenara yatay şerit, `padding-bottom:env(safe-area-inset-bottom)`.
3. Nav sonuna ⛶ düğmesi (44px), `toggleFullscreen()`; `fullscreenchange`'te ikon değişir; desteklenmiyorsa gizli.
4. `#f11-hint`: yalnız masaüstü + ilk açılış; ⛶ tıklanınca kalıcı kapat.
5. Test: 1280×720 ve 375×812'de canvas `clientWidth/Height` == viewport; merkez tıklaması haritaya gider.

Maliyet: ~60 satır CSS, ~15 satır JS, 2 dosya (style + view-switch), yeni dosya yok.
