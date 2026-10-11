ESP32 Wi-Fi Sızma Testi Aracı
El Sıkışma (Handshake) Yakalama
WPA/WPA2-PSK (Kişisel) üzerindeki en yaygın saldırı, el sıkışma (handshake) çerçevelerini dinlemek (sniffing) ve bunlardan PSK'yı (Pre-Shared Key, ağ şifresi olarak bilinir) kırmaktır. WPA/WPA2 el sıkışması sırasında, her iki taraf da daha sonra her iki tarafta aynı PTK'yı (Pairwise Transient Key) hesaplamak için kullanılan parametreleri değiş tokuş eder. Örnek bir WPA/WPA2 el sıkışması aşağıdaki resimde gösterilmiştir:

PTK, kimliği doğrulanmış tek bir oturum için kullanılır ve iletişim bununla şifrelenir (aslında doğrudan PTK'nın kendisiyle değil, PTK'dan farklı anahtarlar türetilir, ancak PTK'nız varsa bunları siz de türetebilirsiniz). Dolayısıyla, PTK'yı hesaplayabilen herkes istasyon (STA) ile erişim noktası (AP) arasındaki iletişimin şifresini de çözebilir. Bu herkese açık olarak iletilen parametrelerin yanı sıra, her iki tarafın önceden bildği ve asla iletilmeyen bir de PMK (Pairwise Master Key) vardır. WPA/WPA2-PSK (Kişisel) için PMK, PSK'ya eşittir; WPA/WPA2-Enterprise için ise PMK, MSK/AAA anahtarından türetilir (bu yazıda ele alınmamıştır). PSK'nın kendisi de daha fazla bölünebilir. Parola (passphrase), bulmamız/kırmamız gereken asıl sirdir. Bu durum aşağıdaki diyagramda görselleştirilmiştir:

Mavi hücreler, el sıkışmayı okuyan herkesin erişebileceği genel kullanıma açık değerlerdir. Turuncu olanlar ise gerçek sırlar/gizli verilerdir.

PTK
PMK - Pairwise Master Key (İkili Ana Anahtar). Her iki taraf da bu değeri önceden bilir ve asla iletilmez.

ANonce - AP tarafından rastgele oluşturulan değer

SNonce - STA tarafından rastgele oluşturulan değer

MAC AP + MAC STA - AP ve STA Wi-Fi ağ arayüzlerinin (NIC) MAC adresleri

PMK/PSK
Passphrase (Parola) - Bu genellikle ağ şifresi, parola, Wi-Fi şifresi vb. olarak bilinir. Bu, tüm WPA/WPA2-PSK'nın dayandığı gerçek ve tek gizli kısımdır. Bunu bilirsek, PTK'yı hesaplayabilir ve trafiğin şifresini çözebilir veya MAC filtreleme gibi başka önlemler alınmamışsa doğrudan ağa katılabiliriz.

SSID - Yaygın olarak Wi-Fi ağ adı olarak bilinir. Bu, çevredeki mevcut ağlar listesinde gördüğünüz insan tarafından okunabilen (olması gerekmese de) dizgedir.

4096 - Bu, parolanın hash'lenme (şifrelenme) sayısıdır. Bu değer tasarım gereği sabittir.

256 - PSK'nın boyutu (bit cinsinden). Bu değer de tasarım gereği sabittir.

WPA/WPA2 el sıkışması EAPoL protokolünü ve bunun EAPoL-Key türündeki paketlerini kullanır. Bu paketler 802.11 veri çerçeveleri içine kapsüllenmiştir. Aşağıdaki çerçeve yapısı dökümüne bakın:

Basitleştirilmiş WPA/WPA2 el sıkışma değişimi, bu yazının başındaki sıra diyagramında görülebilir.

Yukarıdaki açıklamadan, aslında WPA/WPA2 el sıkışmalarının dört mesajına da ihtiyacımız olmadığını görüyoruz. Sadece tüm parametreleri yakalamamız ve Mesaj Bütünlük Koduna (MIC - Message Integrity Code) sahip bütün bir EAPoL paketine sahip olmamız yeterlidir. MIC, yeni hesaplanan PTK kullanılarak (başlangıçta sıfırlanmış MIC alanı ile) tüm EAPoL paketinin şifrelenmesiyle hesaplanır. MIC, ikinci mesajdan (STA'dan gelen ilk mesaj) itibaren mevcuttur. Karşı tarafın mesajın bir saldırgan tarafından sahte olarak üretilmediğini doğrulayabilmesi için pakete sonradan eklenir.

Bu, ağ parolasını tahmin ederek, PSK ve PTK'yı hesaplayarak ve ardından yakalanan EAPoL-Key paketini şifreleyerek kaba kuvvet (brute-force) saldırısı için kullanılabilir. Sonuç, EAPoL-Key'de orijinal olarak bulunan MIC ile eşleşirse, doğru PTK'yı ve parolayı bulmuşuz demektir.

Kimlik Doğrulama Düşürme (Deauthentication) Saldırısı
El sıkışma yakalamanın tek dezavantajı, hedef AP ile herhangi bir el sıkışma gerçekleştiği sırada etrafta olmanız gerekliliğidir. Bu durum el sıkışma yakalamayı öngörülemez hale getirir. İstendiğinde el sıkışmayı tetiklemek için, bu tür saldırılardan önce genellikle kimliği doğrulanmış STA'ları AP'den koparan bir deauthentication saldırısı gerçekleştirilir. Bu durum, kablosuz cihaz üreticilerinin, cihazların kullanıcı haberi olmadan ağdan koparılmaları durumunda otomatik olarak yeniden bağlanmaya çalışması ortak uygulamasıyla birleştirilir. 802.11 standardı, önceden kimliği doğrulanmış oturuma devam etmek istemediğini bildirmek için AP veya STA'nın karşı tarafına deauthentication çerçevesi gönderebileceği bir süreç tanımlar. Bu gerçekleştiğinde, iletişime devam etmek isteyen STA, kimliğini yeniden doğrulamak zorundadır.

PMKID Yakalama
Yapılacaklar listesinde (TBD)