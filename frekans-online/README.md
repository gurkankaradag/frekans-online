# Frekans Online 3.0

İki takımlı, özel lobi koduyla oynanan Türkçe tarayıcı oyunu. 418 hazır kart, özel kart havuzu, genel ve takım sohbeti, canlı ortak kadran, rol denetimli sunucu puanlaması. Orijinal çevrimdışı HTML sürümü de `/offline.html` adresinde korunmuştur.

## Windows / yerel çalışma

1. Node.js 22 veya daha yeni desteklenen sürümü yükleyin: https://nodejs.org/
2. ZIP'i açın. PowerShell veya CMD'yi `frekans-online` klasöründe açın.
3. `npm start` komutunu çalıştırın. Harici npm bağımlılığı yoktur.
4. Chrome veya Edge'den `http://localhost:3000` açın.
5. Bir oyuncu lobi oluşturur, diğer üç oyuncu davet bağlantısı veya 6 karakterli kodla katılır. Tek bilgisayarda denemek için ayrı tarayıcı profilleri/özel pencereler kullanın. Aynı ağdaki bilgisayarlar sunucunun yerel IP adresini ve 3000 portunu kullanabilir (güvenlik duvarı izni gerekebilir). İnternet için yerel adres değil aşağıdaki barındırma yöntemi kullanılır.

## Render üzerinden internet yayını

- GitHub'da yeni bir repo oluşturun; bu klasörün **içindeki** dosyaları repo köküne yükleyin (`server.js` ve `package.json` kökte, `public/` alt klasör olarak kalmalı).
- https://dashboard.render.com/ üzerinde New > Web Service seçin, repo'yu bağlayın.
- Runtime: Node; Build Command: `npm install`; Start Command: `npm start`; Health Check Path: `/health`.
- Instance türü: deneme için Free (varsa), kesintisiz kullanım için uygun ücretli plan. `PORT` otomatik okunur; sunucu `0.0.0.0` adresinde dinler.
- Deploy tamamlanınca panelde görünen gerçek `https://...onrender.com` adresini kullanın. İsterseniz Render'ın Custom Domains ayarıyla kendi alan adınızı bağlayın.
- Alternatif: repo'daki `render.yaml` dosyası Blueprint kurulumu içindir; panelden manuel kurulum yapılıyorsa Blueprint zorunlu değil.

Belgeler: https://render.com/docs/web-services ve https://render.com/docs/free . Ücretsiz servis boşta uyuyabilir; RAM'deki lobiler yeniden başlatmada kaybolur.

## Oyun

4–16 oyuncu (takım başına en az iki aktif oyuncu). Lobi sahibi takım isimlerini, kart destesini ayarlar ve başlatır. İpucu veren sırayla seçilir, hedefi sadece o görür. Aktif takımın belirlenen bir oyuncusu ortak kadranı yönetir, diğerleri takım sohbetinden tartışır. Rakip takımın belirlenen oyuncusu sağ/sol seçer. 10 puan hedefi, 2/3/4 puan bölgeleri, rakip bonusu, geri dönüş ve uzatma kuralları sunucuda hesaplanır. İpucu veren hedefi açtıktan sonra sonuç aşamasına kadar sohbet gönderemez. Lobi sahibi sonraki turu başlatır.

Özel kartlar lobide eklenebilir/silinebilir. İpucu veren hedef açılmadan önce yalnızca o tur kullanılacak bir kart yazabilir. Özel kartlar çevrimiçi lobinin ömrü boyunca saklanır. Çevrimdışı sürümün tarayıcıya kayıt özelliği ayrıdır.

## Mimari / sınırlar

- Node.js yerleşik HTTP sunucusu, HTML/CSS/JavaScript arayüzü. İstemciler her 650 ms'de yetkili durum sorgusu yapar; eylemler POST ile gönderilir. WebSocket gerekmez. Bu ilk sürüm küçük arkadaş gruplarına yöneliktir; yüksek trafik için WebSocket/Redis gibi bir mimari ve yük testi önerilir.
- 192 bit rastgele misafir oturum anahtarı, sunucu tarafı rol/evre denetimleri, hedefin yetkisiz yanıtlardan çıkarılması, sohbet erişim kontrolü, temel hız/boyut sınırları ve kullanıcı metninin HTML kaçışlaması vardır. Harici uygulamalar üzerinden yapılan hileyi engellemez. Davet kodunu bilenler oyun başlamadan katılabilir; hesap sistemi veya parola yoktur.
- Oyun durumu ve mesajlar RAM'de tutulur; tek sunucu örneği çalıştırın. Yeniden başlatma veya 30 dakika hareketsizlik lobiyi siler. Son 160 sohbet mesajı tutulur; kalıcı veritabanı yoktur.
- Tarayıcı yenilenirse aynı sekmedeki misafir oturumu korunur. 15 saniye görülmeyen oyuncu çevrimdışı işaretlenir; sekme arka planda uzun süre kalırsa tarayıcı sorguları yavaşlatabilir. Oyun ortasında yeni oyuncu giremez; lobiye dönüldüğünde katılabilir. Aktif oyuncu koparsa lobi sahibi yeterli oyuncu varsa turu yenileyebilir veya lobiye dönebilir.
- Kamuya geniş çapta yayın öncesi raporlama/moderasyon, gizlilik bildirimi, istismar önleme, yük testi, kalıcı veri ve marka/telif değerlendirmesi yapılmalıdır. Sesli sohbet yoktur.

## Test

`npm test` dört bağımsız misafir oturumuyla gerçek yerel HTTP API'yi test eder: lobi, takım sohbeti izolasyonu, genel sohbet, özel kart, gizli hedef, yetkisiz işlemler, ortak tahmin, puanlama ve sonraki tur. Sunucu ve istemci JavaScript sözdizimi kontrol edildi. Bu üretim ortamına dağıtım testi değildir. Chromium'un yerel adres erişimi test ortamı politikası tarafından engellendiğinden uçtan uca tarayıcı testi tamamlanamadı; yayından önce yukarıdaki adımları dört cihazla deneyin.
