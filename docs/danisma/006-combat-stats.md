# 006 — Savaş Stat Dengesi Danışması (Fable)

`Agent(model=fable)` · Wave B combat core · 2026-09-17 · ~59.5k token / 52 s

Taban (%18 afterArmor floor) matrisi kıpırdatmadı — gerçek "1 hasar" yoktu, dengesizlik ham
stat kaynaklı. Fable'a somut statlar + mevcut matris + motorun counter mekanikleri (brace ×1.5,
şarj, adrenalin) verilip 3 net soru soruldu.

## Girdi (Fable'a giden, tam metin)

> Webband adlı tarayıcı RPG'sinin (Mount&Blade Warband tarzı) birim dengesini kuruyorum. Sen tasarım danışmanısın; kısa ve kararlı cevap ver. Aşağıda GERÇEK veri var.
>
> ## Hasar tipleri (zırh = savunmanın kaçta kaçı uygulanır, mult = ham çarpan)
> - kesici (cut):  armor 1.0, mult 1.0
> - delici (pierce): armor 0.5, mult 0.9
> - ezici (blunt): armor 0.65, mult 0.8, bayıltır (öldürmez, esir alır)
>
> Hasar formülü (tek kapı): landed = max(rawHasar*mult - savunma*armor, rawHasar*mult*0.18); yani zırh bir vuruşu köreltir ama %18 taban hep geçer. rawHasar = birimin attack değeri.
>
> ## Birim statları [tip, hp, hız, saldırı, savunma, dmgType] — üst-kademe (elit) birimler:
> - Nord Baltacısı: piyade, hp80, hız74, atk24, def13, cut  (baltacı, zırh deler)
> - Rodok Kalkanlısı: piyade, hp70, hız58, atk17, def18, cut  (dev kalkan, en yüksek savunma)
> - Rodok Mızraklısı (orta kademe): piyade, hp48, hız56, atk11, def8, pierce (mızrak)
> - Svadya Şövalyesi: SÜVARİ, hp75, hız110, atk22, def15, cut (en güçlü ağır süvari)
> - Kergit Süvarisi: SÜVARİ, hp58, hız115, atk18, def8, cut
> - Kergit Atlı Okçusu: okçu(atlı), hp40, hız108, atk10, def3, pierce
> - Rodok Tatar Yaylısı: okçu, hp48, hız56, atk16, def6, pierce
> - Veagir Baltacısı: piyade, hp62, hız64, atk20, def9, cut
> - Svadya Milisi (orta): piyade, hp45, hız60, atk12, def5, pierce
>
> ## Motorun SAHİP OLDUĞU counter mekanikleri (kodda çalışıyor):
> 1. Anti-süvari brace: duran, pierce(mızraklı) piyade şarj eden ata vurunca hasar ×1.5, ayrıca atı yavaşlatır.
> 2. Süvari şarjı: hızlıyken hasar artar; mızraklıda (couched) daha çok.
> 3. Adrenalin: süvariden yiyen piyade 3sn hız patlaması alır (atı yakalasın diye).
>
> ## Ölçüm (1v1, gerçek motor, 50 dövüş):
> - Nord Baltacısı vs Rodok Kalkanlısı: A %100 kazanıyor (baltacı kalkanlıyı eziyor)
> - Nord Baltacısı vs Svadya Şövalyesi: A %4 (süvari piyadeyi eziyor — hp+hız üstünlüğü)
> - Rodok Mızraklısı vs Nord Baltacısı: A %0
> - Svadya Milisi vs Rodok Kalkanlısı: A %0
> - Kergit Atlı Okçusu vs Rodok Tatar Yaylısı: A %14
>
> Not: mızrak↔süvari eşleşmesi HİÇ ölçülmemiş (PAIRS'te yok). %18 taban eklenince matris neredeyse HİÇ değişmedi çünkü gerçek bir "1 hasar" durumu yoktu; dengesizlik ham stat kaynaklı (süvari hp+hız ile eziyor, mızrak/kalkan çok zayıf).
>
> ## Hedef felsefe (senin daha önce verdiğin range kuralı):
> Aynı kademe iki birim 40–60; bir kademe fark 65–80; iki kademe 85+; KARŞIT tip (mızrak→süvari, ok→hafif) +10.
>
> ## SANA 3 NET SORU (her birine kısa, sayısal, kararlı cevap ver):
> 1. Ölçüm matrisine hangi 4-6 "çapa eşleşmesi" konsun ki taş-kağıt-makas doğru sınansın? (özellikle mızrak vs süvari, süvari vs okçu, aynı kademe piyade vs piyade). Liste ver.
> 2. "Nord Baltacısı vs Svadya Şövalyesi %4" — baltacı piyadesi ağır süvariye karşı ne kazanmalı? Piyade normalde süvariye karşı DEZAVANTAJLI olmalı ama %4 çok mu düşük? Hedef aralık ver.
> 3. Yukarıdaki statlarda hangi SOMUT değişiklikler bu dengeyi kurar? (örn. Svadya Şövalyesi hp75→?, Rodok Kalkanlısı savunma/hp→?, mızraklı atk/hp→?). Az sayıda, cerrahi değişiklik iste — tüm tabloyu elden geçirme. Değişiklik başına tek satır gerekçe.
>
> Kısa yaz. Madde madde.

## Çıktı (Fable'dan gelen, tam metin)

> ## 1. Çapa eşleşmeleri (hedef = A'nın kazanma %)
>
> | # | A vs B | Sınadığı şey | Hedef A |
> |---|---|---|---|
> | 1 | Rodok Mızraklısı vs Kergit Süvarisi | mızrak→hafif süvari (orta vs orta-üst) | 50–60 |
> | 2 | Rodok Kalkanlısı vs Svadya Şövalyesi | elit mızrak→elit ağır süvari (3. sorudaki pierce değişikliğiyle) | 55–65 |
> | 3 | Svadya Şövalyesi vs Rodok Tatar Yaylısı | süvari→okçu | 80–90 |
> | 4 | Kergit Süvarisi vs Kergit Atlı Okçusu | hafif süvari→atlı okçu | 65–75 |
> | 5 | Nord Baltacısı vs Rodok Kalkanlısı | aynı kademe piyade-piyade | 50–60 |
> | 6 | Nord Baltacısı vs Svadya Şövalyesi | kontrol: baltacı→ağır süvari | 30–40 |
>
> Nord vs Veagir Baltacısı'nı (65–75, bir kademe fark) yedek olarak tut. Okçu eşleşmelerinde başlangıç mesafesini sabitle (ör. 3 ok atacak menzil); yoksa ölçüm ok değil yumruk ölçer.
>
> ## 2. Baltacı vs Şövalye
>
> %4 çok düşük. Elit baltacı aynı kademe; süvari "tip üstünlüğü" alır, karşıtı değil. Hedef **%30–40**. %20 altı = süvari spam çözümsüz, %45 üstü = ağır süvari amacını yitirir.
>
> ## 3. Cerrahi değişiklikler (5 satır)
>
> - **Svadya Şövalyesi hp 75→70.** Baltacıya 9/vuruş yiyor; 9→8 vuruş ölür, şarj + hız farkı süvariyi hâlâ ~%65'te tutar.
> - **Rodok Kalkanlısı dmgType cut→pierce.** Mızrak+kalkan zaten Rodok kimliği; Rodok'un elit anti-süvarisi yok, bu satır onu yaratır (brace ×1.5 devreye girer, Şövalye'ye 7.8×1.5≈12/vuruş).
> - **Rodok Kalkanlısı hp 70→78, atk 17→19.** Baltacıya karşı 12 vuruş→13 dayanır, 20 vuruş→14'te öldürür; def 18'e dokunma, 20 olursa orta kademe %18 tabana çakılır.
> - **Rodok Mızraklısı atk 11→13, hp 48→52.** Kergit Süvarisi'ne brace ile 11.6/vuruş → 5 vuruş; kendisi 5 vuruşta ölür, yavaşlatma ile ~%55.
> - **Brace çarpanı 1.5 sabit kalsın.** Yalnız #1 ölçümü %40 altı çıkarsa 1.75'e çek; stat yerine mekanik oynamak tek noktadan düzeltir.
>
> Değişmeyen: Nord Baltacısı, Kergit çifti, Tatar Yaylısı. Atlı Okçu %14 doğru (orta vs elit, bir kademe fark).
