# TUG-QuestDB
## QuestDB nedir?
QuestDB diğer veri tabanlarından çok daha farklı bir yapıya sahiptir ve gelecekte yaygınlaşması muhtemeldir.Geleneksel veritabanlarının aksine, zaman serisi verilerinin benzersiz özelliklerini (çok sayıda satır ekleme, sorgulama ve sıralama) en iyi şekilde ele almak için özel bir mimariye sahiptir. QuestDB, özellikle zaman serisi (time-series) verileri için tasarlanmış, yüksek performanslı, açık kaynaklı bir veritabanıdır. Temel odak noktası, finansal piyasa verileri, IoT sensör okumaları, uygulama metrikleri, telemetri ve olay izleme (log kayıtları) gibi saniyede milyonlarca veri noktasının alındığı ve hızlı bir şekilde sorgulandığı senaryolardır. 


###  QuestDB'nin temel yapı taşları:
- **Kolon Bazlı Depolama (Columnar Storage):** Bu mimaride veriler satır satır değil, sütun sütun saklanır. Geleneksel bir veritabanında bir satırın tüm verileri (örneğin, bir sensörden gelen tarih, sıcaklık, basınç) tek bir blokta saklanır. QuestDB'de ise her bir sütunun verileri ayrı ayrı depolanır. Disk yapısında, her tablo bir klasör olarak saklanır. Her sütun bir dosyadır.  Örnek: "Ortalama sıcaklık nedir?" gibi bir sorgu yaptığınızda, veritabanı sadece temperature.bin dosyasını okur. Tüm satırları (cihazID, nem gibi ilgisiz verileri) okumak zorunda kalmaz. Bu, disk I/O işlemlerini büyük ölçüde azaltır.
- **Zaman Bazlı Bölümleme (Partitioning):** QuestDB, verileri zamana göre otomatik olarak bölümlere (partition) ayırır. Bu, veritabanının en güçlü özelliklerinden biridir. Veriler, belirli zaman aralıklarına göre (örneğin, günlük, aylık veya yıllık) ayrı dosyalarda saklanır. Yani zaman damgasına göre otomatik bölümlere ayrılır (ör. günlük, aylık, yıllık). Bu sayede sorgular sadece ilgili bölümlere bakar.
- **SQL Uyumluluğu:** Sorgular için standart SQL sözdizimi kullanır. Özel bir sorgu dili yoktur, bildiğimiz SQL ile çalışılır. Sadece zaman serisi analizi için özel fonksiyonlar ve sözdizimi ekler(SAMPLE BY, LATEST ON,ASOF JOIN gibi). Bu, onu öğrenmesi ve kullanması kolay bir araç yapar.
- **Yüksek Verim (Throughput) ve Düşük Gecikme (Latency):** Tek bir sunucuda saniyede milyonlarca satırlık veri ekleme kapasitesine sahiptir. Veriler milisaniyeler içinde sorgulanabilir hale gelir. 




### Ana özellikleri şunlardır:
- Çok hızlı yazma ve okuma (milyonlarca kayıt/saniye işleyebilir).
- Kolay entegrasyon (Postgres wire protocol, REST API, InfluxDB line protocol desteği).
- Yeni kayıtlar her zaman sona eklenir. Bu da diskte sıralı yazmaya izin verir → SSD/HDD için maksimum hız.
- Büyük miktarda alım işleme ve verimlilik
- Yüksek performanslı veri çoğaltma ve sıra dışı dizinleme
- Yüksek veri kardinalitesi performans düşüşüne yol açmayacaktır.(Kardinalite, bir kümeyi oluşturan farklı değer sayısını gösteren bir veri niteliğidir. Yüksek kardinaliteli verilere sahip olmak, veri kümesinde çok sayıda benzersiz değer olduğu anlamına gelir.)
- Donanım verimliliği (Sensörler ve Raspberry Pi dahil olmak üzere çok küçük donanımlarda güçlü, maliyet tasarrufu sağlayan performans.)
- 
- 

### En büyük hitleri şunlardır:
- **SAMPLE BY** Verileri bir yıldan bir mikrosaniyeye kadar belirli bir zaman aralığına göre parçalara özetler
- **WHERE IN** Zaman aralıklarını özlü aralıklara sıkıştırmak
- **LATEST ON** Bir tablodaki birden fazla serideki en son değerler için
- **ASOF JOIN** Yakınlığa dayalı olarak bir seri arasındaki zaman damgalarını ilişkilendirmek için; ekstra endekslere gerek yok
- Performansı optimize etmek için karmaşık sorgu sonuçlarını önceden hesaplamak ve otomatik olarak yenilemek için Maddeleştirilmiş Görünümler



<img width="914" height="644" alt="image" src="https://github.com/user-attachments/assets/8e75123d-8d95-4a98-ae7d-13514ea74c97" />




Ancak daha az dayanıklı donanımlarda fark daha da belirginleşiyor; aşağıdaki kıyaslamada da görüldüğü gibi.

Raspberry Pi 5 gibi hafif bir donanımda bile QuestDB, daha güçlü donanımlardaki rakiplerinden daha iyi performans gösteriyor:


<img width="872" height="623" alt="image" src="https://github.com/user-attachments/assets/9a8118f9-00a8-4cb0-8445-07360c99ac19" />

**Resimlerin kaynakçası https://questdb.com/docs/why-questdb/**


### QuestDB'nin tek veritabanı özelliği
QuestDB, her örnek için tek bir veritabanına sahiptir . PostgreSQL ve diğer veritabanı motorlarının aksine, bir örnekte birden fazla veritabanı veya birden fazla şema bulunabilirken, QuestDB'de tek bir ad alanında çalışırsınız.

Varsayılan veritabanının adı'dır qdbve bu, yapılandırma aracılığıyla değiştirilebilir. Ancak, standart bir SQL veritabanının aksine, USE DATABASEkomut vermenize gerek yoktur. Bağlandıktan sonra, hemen sorgulamaya ve veri eklemeye başlayabilirsiniz.





## QuestDB and C#
C# tarafında en yaygın PostgreSQL kütüphanesi Npgsql’dir. QuestDB de Postgres protokolünü konuştuğu için doğrudan kullanılabilir.
En sağlam yöntem → ADO.NET + Dapper.
