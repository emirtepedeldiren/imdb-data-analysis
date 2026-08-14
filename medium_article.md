# Bir Filmin Gişede Tutacağını Önceden Bilebilir miyiz?

---

James Cameron 2009'da Avatar'ı çekmek için 237 milyon dolar harcadı. Film dünya çapında 2,78 milyar dolar hasılat yaptı. Bütçesinin yaklaşık on iki katı. Bugün bunu bir başarı hikâyesi olarak anlatıyoruz, ama 2007'de o parayı onaylayan yöneticiler için ortada hikâye falan yoktu. Sadece bir bahis vardı.

Sinema sektörünün en pahalı sorusu bu: bir film, daha tek karesi çekilmeden, tutup tutmayacağını ele veriyor mu?

Bu soruyu bir veri setine sormaya karar verdim. Elimde Kaggle'dan alınmış 4.803 filmlik devasa bir veri seti vardı; bütçeler, hasılatlar, türler, süreler, vizyon tarihleri. Amacım basitti: önce verinin ne söylediğine bakmak, sonra da bir film vizyona girmeden önce bilinebilecek bilgilerle gişe başarısını tahmin eden bir model kurmak.

Sonuç, beklediğimden farklı bir yere çıktı. Modelin doğruluk oranı hikâyenin en ilginç kısmı değildi. En ilginç kısım, yol boyunca karşıma çıkan dört yanlış sayıydı. Dördü de sessizdi — hiçbiri hata vermedi, hiçbiri uyarı üretmedi. Her biri gayet makul görünen bir rakam verdi ve o rakam yanlıştı.

---

## Elimdeki Veri

TMDB 5000 Movie Dataset, Kaggle üzerinden erişilebilen açık bir veri seti. 4.803 satır, 20 kolon. Her satır bir filmi temsil ediyor ve şunları içeriyor:

- Yapım bütçesi ve dünya çapında hasılat
- Film türleri, süre, vizyon tarihi
- Yapım şirketleri, orijinal dil
- TMDB kullanıcı puanı, oy sayısı ve popülerlik skoru

Veri 1916'dan 2017'ye uzanıyor, ama ağırlığı belirgin şekilde yakın döneme kaymış: filmlerin yaklaşık yüzde 73'ü 2000 sonrasına ait. Yüzde 94'ü de İngilizce. Yani teknik olarak "dünya sineması" değil, pratikte Hollywood'a bakıyoruz.

İlk iş olarak eksik değerlere baktım. Sonuç fena görünmüyordu: film sitesi adresi kolonunun yüzde 64'ü boştu, slogan kolonunun yüzde 18'i eksikti, birkaç filmin süresi kayıptı. Bütçe ve hasılat kolonlarında ise hiç eksik değer yoktu.

Rahatlayıp devam ettim. Etmemeliydim.

---

## Veriler Bana Yalan Söyledi

Bütçe kolonunda eksik değer olmamasının sebebi, verinin eksiksiz olması değildi. Eksik değerlerin boş bırakılmak yerine sıfır yazılmış olmasıydı.

Sayınca ortaya çıkan tablo şuydu: 1.037 filmin bütçesi ve 1.427 filmin hasılatı sıfır görünüyordu. Bir filmin gerçekten sıfır dolara çekilmesi mümkün değil. Bu satırlar "bilinmiyor" demekti, ama kod bunu "sıfır" olarak okuyordu.

Farkın büyüklüğünü görmek için ortalama bütçeyi iki şekilde hesapladım. Sıfırlar dahil edildiğinde ortalama 29 milyon dolar çıkıyordu. Onları eksik kabul edip dışarıda bıraktığımda 37 milyona yükseldi. Aradaki fark yüzde 28.

Bunu fark etmeseydim, sonrasında yazdığım her cümle sistematik olarak yanlış olacaktı. Ve hiçbir yerde kırmızı bir uyarı çıkmayacaktı. Verinin bana söylediği ilk yalan buydu; arkasından üç tane daha gelecekti.

Buradan çıkardığım ders şu oldu: eksik veriyi kontrol etmek, eksik değer sayısına bakmakla bitmiyor. Sayısal kolonlarda sıfırların, kategorik kolonlarda "Unknown" ya da "NaN" gibi değerlerin de aslında "veri yok" anlamına gelebileceğini akılda tutmak gerekiyor.

Bütçesi veya hasılatı bilinmeyen filmleri çıkardığımda elimde 4.803 filmden 3.215'i kaldı. Neredeyse üçte birini kaybettim ve bu kaybın rastgele olmadığını biliyorum: finansal bilgi genelde büyük stüdyo yapımları için kayıtlı tutuluyor, küçük bağımsız filmler kayıtlardan düşüyor. Yani elimdeki veri, sinemanın tamamını değil, görünür kısmını temsil ediyor.

---

## "Başarılı Film" Neye Denir?

Modelin tahmin edeceği şeyi tanımlamam gerekiyordu. İlk akla gelen "hasılatı bütçesinden yüksekse kâr etmiştir" yaklaşımı yanıltıcıydı, çünkü veri setindeki bütçe rakamı yalnızca yapım maliyetini içeriyor. Pazarlama ve dağıtım masrafları o rakamın içinde yok ve bunlar küçük kalemler değil.

Sektörde kabaca şöyle bir kural işliyor: bir film başabaş noktasını, yapım bütçesinin yaklaşık iki katında geçer. Ben de bu eşiği kullandım. Hasılatı bütçesinin en az iki katı olan filmleri "başarılı", olmayanları "başarısız" saydım.

Bu tanımla elimdeki 3.215 filmin yüzde 56,1'i başarılı sınıfına girdi. Dengeli bir dağılım, ki bu ilerisi için önemli olacak.

---

## Ortalama Diye Bir Şey Yok

Yatırım getirisine bakmaya başladığımda ilk hesapladığım sayı ortalamaydı: 11,13. Yani filmler ortalamada bütçelerinin on bir katını kazanıyordu.

Bu rakam saçmaydı. Medyana baktığımda 2,30 çıkıyordu.

Aradaki uçurumun sebebini bulmak zor olmadı. 2007 yapımı *Paranormal Activity* 15 bin dolara çekilmiş ve 193 milyon dolar hasılat yapmıştı. Getirisi 12.890 kat. Yalnızca bu film, 3.215 filmlik veri setinin ortalamasını 7,1'den 11,1'e çıkarıyordu. Onu tamamen çıkarsam bile ortalama hâlâ medyanın üç katıydı — çünkü arkasında sıra sıra benzerleri vardı.

Bu, ikinci yalan noktasıydı ve sonrasındaki bütün analizin şeklini belirledi: uç değerlerin bu kadar baskın olduğu bir dağılımda ortalama hiç kimseyi temsil etmiyor. Yazının geri kalanında göreceğiniz her getiri rakamı medyandır.

---

## Orta Sınıfın Ölümü

Bütçe ile hasılat arasında güçlü bir ilişki var; ikisi arasındaki korelasyon 0,70. Yani pahalı filmler gerçekten daha çok kazanıyor — burada sürpriz yok.

(Birazdan korelasyon ölçmenin ne kadar aldatıcı olabileceğini göreceğiz. Bu rakama güvenebiliriz: sıralama bazlı ölçümle de 0,67 çıkıyor, yani ikisi birbirini doğruluyor.)

Sürpriz, kazanç yerine kârlılığa baktığımda ortaya çıktı. Filmleri bütçelerine göre yaklaşık eşit büyüklükte beş gruba ayırdım ve her grubun başarı oranını hesapladım. Beklediğim şey düz bir eğilimdi: ya pahalı filmler daha güvenli çıkacaktı, ya ucuz filmler.

Çıkan şekil ikisi de değildi.

![Bütçe grubuna göre başarı oranı](figures/05_butce_gruplari.png)
*Filmler bütçelerine göre beş gruba ayrıldı; her grupta 580 ile 725 arasında film var.*

Bütçesi 8 milyon doların altındaki filmlerin yüzde 66'sı parasını çıkarıyor — listenin en tepesi. Bütçe yükseldikçe oran düşüyor ve 20-35 milyon dolar bandında yüzde 48'e kadar iniyor. Sonra beklenmedik bir şey oluyor: yükselmeye başlıyor. 65 milyon doların üzerindeki yapımlar yüzde 59,5 ile ortalamanın üzerine geri çıkıyor.

İlişki düz bir çizgi değil, U şeklinde.

Yani devasa bütçeli yapımlar sanıldığı kadar riskli değil; onlar zaten bilinen markalar, hazır seyirci kitleleri ve dev pazarlama makineleriyle geliyor. Gerçekten tehlikeli bölge ortası. Ne bağımsız filmin düşük maliyet avantajına sahip, ne blockbuster'ın pazarlama gücüne. Sektörde yıllardır "orta bütçeli filmin ölümü" diye konuşulan şey, veride açıkça görünüyor.

Bu bulguyu aklınızın bir köşesinde tutun. Birazdan modelin neden tuhaf davrandığını açıklayacak.

---

## Korku Filmlerinin Sessiz Üstünlüğü

Türlere göre getiriye baktığımda tablo netleşti.

![Türlere göre medyan yatırım getirisi](figures/06_tur_roi.png)

Listenin tepesinde 3,12 katlık getirisiyle müzik filmleri var, ama örneklem 111 filmle nispeten küçük. Asıl dikkat çekici olan ikinci sıra: **korku filmleri.** 329 filmle sağlam bir örneklem, medyan bütçe yalnızca 14 milyon dolar, medyan getiri 2,90 kat ve en önemlisi — yüzde 67'lik hit oranıyla tüm listenin en tutarlısı.

Korku türü ucuza üretiliyor, sadık bir seyirci kitlesi var ve yıldız oyuncuya ihtiyaç duymuyor. *Paranormal Activity* ile *The Blair Witch Project*in aynı listede olması tesadüf değil.

Diğer uçta ise western ve tarihî filmler duruyor. İkisinin de medyan getirisi 2'nin altında — yani tipik bir western, sektör kuralına göre başabaş noktasını bile geçemiyor.

Bir not: bunlar medyan değerler, garanti değil. Yüksek getirili bir türde çekilen kötü bir film yine batar.

---

## Eylül Ayı Laneti

En beklenmedik bulgu çıkış takviminde saklıydı.

![Çıkış ayına göre gişe başarısı](figures/07_ay_hit_orani.png)

Haziranda vizyona giren filmlerin yüzde 67,4'ü parasını çıkarıyor. Eylülde bu oran yüzde 43,3'e düşüyor. Aradaki fark 24 puan — çekim öncesi bilinebilen değişkenler arasında gördüğüm en büyük etki. Bütçe grubunun yarattığı fark 18 puan, türün yarattığı fark 20 puanda kalıyor.

Yaz tatili ve yılbaşı sezonunun güçlü olması beklenen bir şey. İşin ilginç tarafı şu: eylül, veri setinde **en çok film çıkan ay.** 383 film. Yani stüdyolar en kötü aya en çok filmi yığıyor.

Bunun sektörde bir adı var. Yaz blockbuster'ları bitmiş, ödül sezonu henüz başlamamıştır; eylül, stüdyoların güvenmediği filmleri "boşalttığı" aydır. Grafikteki çukur, bir tercih değil bir itirafın izi.

---

## İyi Film mi, Çok Kazandıran Film mi?

Merak ettiğim bir soru daha vardı: iyi film yapmak para kazandırıyor mu?

TMDB puanı ile yatırım getirisi arasındaki korelasyona baktım. Sonuç tam olarak 0,000 çıktı. Sıfır. Aralarında hiçbir ilişki yok gibiydi.

Neredeyse "sinemada kalite ile kâr birbirinden bağımsızdır" diye yazacaktım. Bu, üçüncü yalandı.

Sorun kullandığım araçtaydı. Pearson korelasyonu ham değerlerle çalışır ve uç değerlere karşı savunmasızdır — *Paranormal Activity*nin 12.890'lık getirisi, tek başına katsayıyı eziyordu. Sıralamalarla çalışan Spearman korelasyonuna geçtiğimde sonuç **0,335** oldu: orta güçte, pozitif ve anlamlı bir ilişki.

Veriye grup grup baktığımda zaten apaçıktı:

![Puana göre getiri ve başarı oranı](figures/08_puan_vs_roi.png)

5'in altında puan alan filmlerin yalnızca yüzde 31'i parasını çıkarıyor ve medyan getirileri 1'in altında — yani ortalama bir kötü film, yatırımcısına zarar ettiriyor. 7,5 üzeri puan alan filmlerde ise hit oranı yüzde 82'ye, medyan getiri 4,90 kata fırlıyor. Aradaki artış kusursuz şekilde kademeli.

Yani iyi film yapmak gerçekten para kazandırıyor. Ama bu bilginin bir sorunu var: bir filmin puanı, ancak insanlar onu izledikten sonra oluşuyor.

Bu detay, projenin en kritik kararına götürdü beni.

---

## Modelden Sakladıklarım

Elimde tahmin gücü çok yüksek üç değişken vardı: popülerlik skoru, oy sayısı ve kullanıcı puanı. Bunları modele eklesem doğruluk oranı ciddi biçimde yükselirdi.

Eklemedim.

Sebebi şu: üçü de film vizyona girdikten sonra oluşuyor. Bir filmin kaç kişi tarafından oylandığını bilmek, aslında o filmin çok izlendiğini bilmek demek. Bu bilgiyi modele vermek, cevabı soruyla birlikte vermekle aynı şey. Makine öğrenmesinde buna **veri sızıntısı** deniyor.

Böyle bir model kâğıt üzerinde parlak görünür. Ama modelin gerçekte kullanılacağı an, bir yapımcının senaryoya bakıp "bunu çekelim mi?" diye sorduğu andır — ve o anda bu bilgilerin hiçbiri mevcut değildir.

Bu yüzden modele sadece çekim öncesi bilinebilecek şeyleri verdim: bütçe, süre, çıkış yılı ve ayı, yapım şirketi sayısı, filmin İngilizce olup olmadığı ve türü. Toplam 25 değişken.

Karşılaştırma çizgim de basitti: her filme koşulsuz "bu tutar" diyen bir tahminci yüzde 56,1 doğruluk elde ederdi. Model bunu geçemezse hiçbir işe yaramıyor demekti.

İki model kurdum. Lojistik regresyon yüzde 59,3'te kaldı. Random Forest yüzde 65,9'a çıktı — baseline'ın yaklaşık on puan üzeri.

Sevindim. Erken sevinmişim.

---

## Model Bana İyi Haber Verdi, İnanmadım

Tek bir test setine güvenmemek gerektiğini biliyordum, o yüzden beş katlı çapraz doğrulama çalıştırdım. Sonuç yüzde 55,4 çıktı.

Baseline'ın altında. Test setinde yüzde 66, çapraz doğrulamada yüzde 55. Bu kadar büyük bir fark olamazdı — dördüncü yalanla karşı karşıyaydım.

Sebebi bulmak biraz sürdü. Scikit-learn'den varsayılan ayarlarla çapraz doğrulama istediğinizde, veriyi **karıştırmadan** bölüyor. Bu veri setinde ölümcül bir hata, çünkü dosya kabaca bütçeye göre sıralı: ilk satırlarda Avatar gibi 200 milyon dolarlık yapımlar var, sonlara doğru 15 bin dolarlık *Paranormal Activity*. Karıştırmadan bölünce her kat tamamen farklı bir bütçe dünyasından oluşuyor ve model hiç görmediği bir evrende sınava sokuluyordu.

Çözüm tek bir ayardı: bölmeden önce veriyi karıştırmak. Doğru bölünmeyle sonuç yüzde 62,0'ye çıktı, standart sapma yüzde 1,2. Bu aynı zamanda test setindeki yüzde 65,9'un biraz iyimser olduğunu gösteriyordu. Dürüst rakam yüzde 62.

![Karışıklık matrisi ve ROC eğrisi](figures/10_model_sonuclari.png)

---

## Modelin Bana Öğrettiği Şey

Random Forest'ın hangi değişkenlere ağırlık verdiğine baktığımda çıkış yılı ve bütçe başa baş gidiyordu; ikisi birlikte toplam önemin yaklaşık yüzde 40'ını oluşturuyordu. Tür bilgisinin katkısı şaşırtıcı derecede küçüktü — hiçbir tür tek başına yüzde 3'ü geçmiyordu.

Asıl kafa karıştıran şey lojistik regresyonun katsayılarındaydı. Bütçe değişkeninin katsayısı **negatifti** — model "bütçe arttıkça başarı ihtimali düşer" diyordu. Oysa az önce bunun doğru olmadığını görmüştük: en pahalı grup, ortalamanın üzerinde performans gösteriyordu.

Çelişki değildi. Doğrusal bir modelin sınırıydı.

Lojistik regresyon yalnızca düz çizgi çizebilir. U şeklindeki bir ilişkiye düz çizgi uydurmaya çalıştığınızda, aşağı eğimli bir çizgi elde edersiniz — model U'nun yalnızca sol yarısını görebiliyor, sağa doğru kalkan kısmını ıskalıyordu. Ağaç tabanlı olduğu için bu şekli öğrenebilen Random Forest'ın altı puan öndeliğinin asıl sebebi buydu.

Modeli hayali senaryolarda çalıştırdığımda bunu doğruladı. 40 milyon dolarlık, eylülde vizyona girecek bir gerilim filmine yüzde 58,1 şans veriyordu. Aynı filmi hazirana aldığınızda yüzde 63,5'e çıkıyordu. 20 milyonluk bir korku filmi yüzde 70,4 alıyordu. 200 milyonluk bir aksiyon-macera yapımı ise yüzde 73,4 ile en yüksek skoru.

U eğrisinin iki ucu da kazanıyor. Model bunu kendi başına öğrenmişti.

---

## Geriye Ne Kaldı

Model, hiçbir şey bilmeyen bir tahmincinin yaklaşık altı puan önüne geçebiliyor. Etkileyici bir rakam değil ve olmasını da beklememeliyiz.

Gişe başarısı; oyuncu kadrosu, yönetmenin adı, pazarlama bütçesinin büyüklüğü, o hafta vizyona giren rakipler, ilk eleştirilerin tonu ve saf şansın birleşimiyle belirleniyor. Elimdeki 25 değişken bu denklemin ancak küçük bir kısmını yakalıyor.

Popülerlik ve oy sayısını modele koysaydım çok daha yüksek bir doğruluk oranı yazabilirdim bu yazıya. Ama o model, gerçekten karar verilmesi gereken anda hiçbir işe yaramazdı. Yüksek skor, iyi model demek değil.

Bu projeden aklımda kalan asıl şey ise şu dört rakam oldu:

- Eksik değer kontrolü "hiç eksik yok" dedi — eksikler sıfır olarak saklanıyordu
- Ortalama getiri 11 kat dedi — tek bir filmin etkisiydi, gerçek değer 2,3'tü
- Pearson korelasyonu 0,000 dedi — doğru ölçümle 0,335 çıktı
- Çapraz doğrulama yüzde 55 dedi — veriyi karıştırmayı unutmuştum

Dördü de sessizce yanlıştı. Kod her seferinde çalıştı, hiçbir uyarı vermedi, gayet makul bir sayı üretti. Veri biliminde asıl iş kodu çalıştırmak değilmiş; çıkan sayıya "bu gerçekten mantıklı mı?" diye sormakmış.

### Bu Analizin Sınırları

- Finansal bilgisi olan 3.215 film üzerinden çalıştım; küçük bağımsız yapımlar eksik temsil ediliyor
- Bütçeler enflasyona göre düzeltilmedi, 1916 ile 2016 dolarları aynı kabul edildi
- Analiz ettiğim filmlerin yüzde 96'sı İngilizce, dolayısıyla sonuçlar esas olarak Hollywood'u anlatıyor
- İki katlık başabaş kuralı bir yaklaşımdır, kesin bir muhasebe değil

---

**Kaynaklar ve kod**

Analizin tamamı, tüm grafikler ve modelin kodu Google Colab notebook'unda çalıştırılabilir durumda:
https://colab.research.google.com/github/emirtepedeldiren/imdb-data-analysis/blob/master/imdb_analizi_colab.ipynb

Proje deposu: https://github.com/emirtepedeldiren/imdb-data-analysis

Veri seti: [TMDB 5000 Movie Dataset — Kaggle](https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata)

*Bu yazı, Huawei Student Developers Veri Bilimi ve Makine Öğrenmesi Bootcamp'i final projesi kapsamında hazırlanmıştır.*
