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


!! **InfluxDB**, InfluxData şirketi tarafından geliştirilen bir zaman serisi veritabanıdır. Operasyon izleme, uygulama ölçümleri, Nesnelerin İnterneti sensör verileri ve gerçek zamanlı analiz gibi alanlarda zaman serisi verilerinin depolanması ve alınması için kullanılır. Ayrıca Grafitten veri işleme desteği de vardır.



**QuestDB'de yüksek kardinalite ile başa çıkma**
Aslında, yüksek kardinalitede asıl zorluk veritabanlarının verileri depolama ve işleme biçiminden kaynaklanmaktadır. Örneğin, InfluxDB ve diğer bazı zaman serisi veritabanlarında, her zaman serisi kendi dosya grubunda ayrı ayrı saklanır. Aynı tabloda binlerce veya milyonlarca zaman serisi depolandığında, yazma ve tam tarama sorgu performansı önemli ölçüde düşer. Bunun nedeni, bu yüksek kardinaliteli senaryolarda disk yazma ve okuma işlemlerinin çok daha az sıralı örüntüye sahip olmasıdır. Sonuç olarak, bellek gereksinimleri benzersiz zaman serisi sayısıyla birlikte katlanarak artar.

QuestDB, her sütunun kendi yerel biçiminde ayrı ayrı depolandığı yoğun, sütun tabanlı bir depolama modeli kullanır . Tüm satırlar zamana göre sıralanır ve sıralı örüntüleri korumak için zaman tabanlı bölümlere ayrılır. Bu, QuestDB'nin yüksek kardinalite de dahil olmak üzere tüm senaryolarda iyi bir performans seviyesini korumasını sağlar.


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

Varsayılan veritabanının adı qdb'dir ve bu, yapılandırma aracılığıyla değiştirilebilir. Ancak, standart bir SQL veritabanının aksine, USE DATABASE komut vermenize gerek yoktur. Bağlandıktan sonra, hemen sorgulamaya ve veri eklemeye başlayabilirsiniz.


## Şema oluşturma:
**Önerilen yaklaşım**
Bir şema oluşturmanın en kolay yolu Web Konsolunu kullanmak veya şu SQL komutlarını göndermektir:
- REST API ( CREATE TABLE ifadeler)
- PostgreSQL kablolu protokol istemcileri


**Accessing the Web Console**
Web Console will be available at http://[server-address]:9000. When running locally, this will be http://localhost:9000.
<img width="1061" height="602" alt="image" src="https://github.com/user-attachments/assets/861a55ca-68d7-41e0-bfb8-555d08922756" />
### Web console ekranı bölümleri ve işlevleri
- **Code Editor:** Kod Düzenleyicisi, sözdizimi vurgulama, otomatik tamamlama ve hata izleme gibi özelliklerle SQL sorguları yazıp çalıştırabileceğiniz yerdir. Seçime göre sorgu yürütme, çoklu sorgu yürütme ve sorgu planlamayı destekler.
- **Metrics View:** Metrik Görünümü, QuestDB örneğiniz için gerçek zamanlı izleme ve telemetri yetenekleri sağlar. Veritabanı performansını, WAL işlemlerini ve tabloya özgü metrikleri izlemek için etkileşimli grafikler ve araçlar görüntüler.
- **Schema Explorer:** Şema Gezgini , tabloları ve somutlaştırılmış görünümleri keşfetmek için bir gezinme merkezidir. Veri türlerini içeren sütunlar, depolama yapılandırması (bölümlendirme ve WAL durumu) ve somutlaştırılmış görünümler için temel tablolar dahil olmak üzere her veritabanı nesnesi hakkında ayrıntılı bilgi sağlar.
- **Result Grid:** Sonuç Tablosu, sorgu sonuçlarınızı veri gezinme, dışa aktarma ve görselleştirme özellikleriyle etkileşimli bir tablo biçiminde görüntüler.
- **Query Log:** Sorgu Günlüğü, sorgu yürütme durumunu ve performans ölçümlerini izleyerek gerçek zamanlı geri bildirim sağlar ve son işlemlerin geçmişini tutar. Sorgularınızı optimize etmenize yardımcı olmak için yürütme sürelerini, satır sayılarını ve ayrıntılı hata bilgilerini gösterir.
- **Import CSV** CSV İçe Aktarma arayüzü , otomatik şema algılama, esnek yapılandırma seçenekleri ve ayrıntılı ilerleme takibi ile CSV dosyalarını QuestDB'ye yüklemenize ve içe aktarmanıza olanak tanır. İçe aktarma süreci üzerinde tam kontrole sahip olarak yeni tablolar oluşturabilir veya mevcut tablolara ekleme yapabilirsiniz.


**GRAFANA**: Verileri görselleştirmek ve zaman serisi veri analizini etkinleştirmek için kullanılan popüler bir gözlemlenebilirlik ve izleme uygulamasıdır. 


## Rest API
QuestDB REST API, standart HTTP özelliklerine dayanır ve hazır HTTP istemcileri tarafından anlaşılır. QuestDB ile etkileşim kurmanın basit bir yolunu sunar ve çoğu programlama diliyle uyumludur. API işlevleri URL'ye tam olarak bağlıdır ve argüman olarak sorgu parametrelerini kullanır.

**Mevcut yöntemler**
### /imp (Import data) for importing data from .CSV files.
Tablo biçimindeki metin verilerini doğrudan bir tabloya aktarır. İsteğe bağlı başlıklarla CSV, TAB ve boru ( ) ile ayrılmış girdileri destekler . Veri boyutu konusunda herhangi bir kısıtlama yoktur. Veri türleri ve yapıları, ek yapılandırmaya gerek kalmadan otomatik olarak algılanır.

**Örnek kullanım:**
```bash
curl.exe -F "data=@C:\Users\hilal\OneDrive\Masaüstü\QuestDbDemo\weather.csv" "http://localhost:9000/imp?overwrite=true&name=new_table"

#User defined schema
curl.exe -F "schema=[{\"name\":\"temp\",\"type\":\"DOUBLE\"},{\"name\":\"humidity\",\"type\":\"INT\"},{\"name\":\"city\",\"type\":\"STRING\"}]" -F "data=@C:\Users\hilal\OneDrive\Masaüstü\QuestDbDemo\weather.csv" "http://localhost:9000/imp?overwrite=true&name=new_table"

```


### /exec to execute a SQL statement.
SQL INSERT Query, Giriş noktası bir SQL sorgusu alır ve sonuçları JSON olarak döndürür. Bunu hızlı SQL eklemeleri için de kullanabiliriz, ancak SQL enjeksiyonu sorunlarından kaçınmak için gerekli olan parametreli sorgular için destek olmadığını unutmayın. Yüksek performanslı eklemelere ihtiyacınız varsa **InfluxDB Satır Protokolünü** tercih edin.

**Örnek kullanım:**


```bash
# Tablo Oluşturma (CMD)
curl.exe -G ^
  --data-urlencode "query=CREATE TABLE IF NOT EXISTS trades(name STRING, value INT)" ^
  "http://localhost:9000/exec"


# Satır Ekleme
curl -G ^
  --data-urlencode "query=INSERT INTO trades VALUES('abc', 123456)" ^
  http://localhost:9000/exec
```

  
### /exp to Export Data.
Bu uç nokta, URL kodlu sorguları geçirmenize olanak tanır ancak istek gövdesi, JSON'un aksine kaydedilip yeniden kullanılmak üzere tablo biçiminde döndürülür. Veritabanını bir SQL select sorgusu ile sorgulamaya ve sonuçları CSV olarak almaya olanak tanır. Sonuçları JSON formatında almak için /exec kullanırız. 

```bash
# Önce weather tablosu oluşturulabilir csv'den:

curl.exe -F "data=@C:\Users\hilal\OneDrive\Masaüstü\QuestDbDemo\weather.csv" "http://localhost:9000/imp?overwrite=true&name=weather"

# Ardından dışa aktarma işlemi yapılabilir:

curl.exe -G ^
  --data-urlencode "query=SELECT temp, humidity, city FROM weather" ^
  --data-urlencode "limit=5" ^
  "http://localhost:9000/exp"

```


### Authentication(RBAC(Role-based Access Control))

#### REST API supports two authentication types:
- HTTP basic authentication
- Token-based authentication

İlk kimlik doğrulama türü çoğunlukla web tarayıcıları tarafından desteklenir. Ancak, kullanıcı kimlik bilgilerini bir başlıkta programatik olarak da uygulayabilirsiniz.(Authorization: Basic)


```csharp

# Veritabanının çalıştığını ve SQL sorgularını alıp cevap verdiğini gösterir. Basic Auth kullanıyoruz. Yani kullanıcı adı ve şifreyi base64 ile encode edip HTTP header’a ekliyoruz.

curl -G --data-urlencode "query=SELECT 1;" ^
    -u "my_user:my_password" ^
    http://localhost:9000/exec
```

İkinci kimlik doğrulama türü, bir REST API belirtecinin bir başlıkta belirtilmesini gerektirir  (Authorization: Bearer header)
```charp

# Kullanıcı adı ve şifre yerine tek seferlik veya uzun ömürlü token ile yetki veriyor

curl -G --data-urlencode "query=SELECT 1;" ^
    -H "Authorization: Bearer qt1cNK6s2t79f76GmTBN9k7XTWm5wwOtF7C0UBxiHGPn44" ^
    http://localhost:9000/exec

```

## PostgreSQL kablolu protokol istemcileri
### PostgreSQL ve PGWire

QuestDB, veri girişi için Postgres Wire Protokolünü (PGWire) destekler. **C# ile kullanacağımız zamanlarda da PGWire ile bağlanacağız**. PGWire (Postgres Wire Protokolü) PostgreSQL'in istemci-sunucu iletişim protokolüdür. Sorgulama ve veri çıkışı için QuestDB, PostgreSQL protokolüyle uyumludur. Bu, QuestDB ile favori PostgreSQL istemcinizi veya sürücünüzü kullanabileceğiniz anlamına gelir. PGWire arayüzü, öncelikle QuestDB'den veri sorgulamak için önerilir.

**Not:** PostgreSQL depolama modeli QuestDB'den temel olarak farklıdır. Sonuç olarak PostgreSQL için var olan bazı özellikler QuestDB'de bulunmamaktadır.


### PGWire İstemcisine Genel Bakış:
QuestDB, istemcilerin PostgreSQL istemci kitaplıklarını kullanarak QuestDB'ye bağlanmasını sağlamak için PostgreSQL kablolu protokolünü (PGWire) kullanır. Bu, mevcut PostgreSQL istemcilerini ve kitaplıklarını kullanmanıza olanak tanıdığı için QuestDB'yi kullanmaya başlamak için harika bir yoldur. 

#### Sorgulama ve veri arayüzü
PGWire arayüzü, öncelikle QuestDB'den veri sorgulamak için önerilir. Veri alımı, özellikle yüksek verimli senaryolar için QuestDB, InfluxDB Hat Protokolü'nü (ILP) destekleyen istemcilerinin kullanılmasını önerir . Bu istemciler, hızlı veri girişi için optimize edilmiştir.


#### Zaman dalgası işleme
QuestDB, tüm zaman damgalarını dahili olarak UTC(Coordinated Universal Time) olarak depolar . Ancak, zaman damgalarını PGWire protokolü üzerinden iletirken QuestDB bunları "TIMESTAMP WITHOUT TIMEZONE" olarak gösterir. Bu durum, istemci kütüphanelerinin bu zaman damgalarını varsayılan olarak kendi yerel saat dilimlerinde yorumlamasına ve bu da olası karışıklığa veya hatalı veri gösterimine yol açabilir. Dil özelindeki kılavuzlarımız, istemcinizin bu zaman damgalarını UTC olarak doğru şekilde yorumlayacak şekilde nasıl yapılandırılacağına dair ayrıntılı örnekler sunar.

Mevcut davranış şu an iyileştirilmeye çalışılıyor. Bu arada, zaman damgalarının tutarlı bir şekilde işlenmesini sağlamak için istemci kitaplığınızdaki saat dilimini UTC olarak ayarlamanızı öneririz.

**QuestDB CSV Import – Timestamp ve Partition Kullanımı**

#### 1. CSV Dosyası

CSV’de timestamp kolonunu **ISO8601 UTC formatında** veya **epoch milisaniye** ile sağlamalısınız:

```csv
ts,temp,humidity,city
2025-09-16T10:00:00Z,24.5,40,Antalya
2025-09-16T11:00:00Z,26.1,38,Antalya
2025-09-16T12:00:00Z,27.3,35,Antalya
```
- Not: Z UTC (Coordinated Universal Time) anlamına gelir.

#### 2. CSV’yi QuestDB’ye Import Etme

- /imp endpoint’i ile import ederken timestamp ve partitionBy parametrelerini kullanın:
curl.exe -F "data=@C:\QuestDbDemo\weather.csv" "http://localhost:9000/imp?overwrite=true&name=weather&timestamp=ts&partitionBy=MONTH"

#### BÖYLECE:
- QuestDB timestamp’leri her zaman UTC olarak depolar

- İstemci tarafında timestamp’leri UTC olarak yorumlayın

- Böylece saat farkı veya yanlış veri gösterimi olmaz


#### Yalnızca İleri İmleçler
QuestDB'nin imleçleri yalnızca ileriye yöneliktir ve bu, PostgreSQL'in kaydırılabilir imleç desteğinden (çift yönlü gezinme ve rastgele satır erişimi sağlar) farklıdır. QuestDB ile sorgu sonuçlarında baştan sona sırayla gezinebilirsiniz, ancak geriye gidemez veya belirli satırlara atlayamazsınız. Kaydırılabilir türler için açık DECLARE CURSOR ifadeleri veya ters yönde getirme gibi işlemler (örneğin, Çalışma Alanı BACKWARD) desteklenmez.
Bu sınırlama, kaydırılabilir imleç özelliklerine dayanan istemci kitaplıklarını etkileyebilir. Örneğin, Python'ın psycopg2 sürücüsü bu tür işlemleri denediğinde sorunlarla karşılaşabilir. En iyi uyumluluk için sürücüleri seçin veya mevcut sürücüleri, Python'ın asyncpg sürücüsü gibi yalnızca ileri imleçleri kullanacak şekilde yapılandırın.




### Kullanım Senaryoları
- Time-series analytics için QuestDB kullanımı

- Mevcut PostgreSQL tool'ları ile QuestDB'ye erişim

- High-throughput data ingestion ve analiz



### List of supported features:
- Sorgulama (tüm tipler hariç BLOB)
- Bağlama parametreleriyle hazırlanmış ifadeler
- INSERT bağlama parametreli ifadeler
- UPDATE bağlama parametreli ifadeler
- DDL yürütme
- Toplu ekler
- Düz kimlik doğrulama


### List of unsupported features
- SSL
- Remote file upload (COPY from stdin)
- DELETE statements
- BLOB transfer


### CRUD -> create/read/update/delete  -> Yukarıdaki kendi sitesinden alıntı özellik destekleri var.

- **CREATE ve READ var**
- **UPDATE**: QuestDB UPDATE desteklemez. Tek yol: yanlış kaydı silip (veya tabloyu truncate edip), doğru veriyi yeniden insert etmek. Yani klasik anlamda update yok
- **DELETE**: QuestDB’de DELETE sadece timestamp (designated timestamp) alanı ile çalışır. DELETE WHERE value = 123.45 gibi non-time sütunlarında çalışmaz. Kısaca, var ama sadece ts üzerinden veya TRUNCATE ile diyebiliriz.



## ILP Protokolü ile Şema otomatik oluşturma
Influx Line Protocol (ILP) kullanıldığında , QuestDB gelen verilere göre otomatik olarak tablolar ve sütunlar oluşturur.Bu özellik, InfluxDB'den geçiş yapan veya InfluxDB istemci kütüphaneleri ya da Telegraf gibi araçlar kullanan kullanıcılar için kullanışlıdır , çünkü önceden tanımlanmış şemalar olmadan verileri doğrudan QuestDB'ye gönderebilirler. Ancak bunun bazı sınırlamaları vardır:
- QuestDB, otomatik olarak oluşturulan tablolara ve sütunlara varsayılan ayarları uygular (örneğin, bölümlendirme, sembol kapasitesi ve veri türleri).
- Bölümlemeyi veya sembol kapasitesini daha sonra değiştiremezsiniz .
- Veri türünü otomatik olarak oluşturamazsınız IPv4. Bir IP adresini dize olarak göndermek bir sütun oluşturacaktır VARCHAR.
Sütun otomatik oluşturmayı yapılandırma yoluyla devre dışı bırakabilirsiniz . QuestDB, verileri almak için InfluxDB Hat Protokolünü uygular. InfluxDB Hat Protokolü yalnızca veri alımı içindir Her ILP istemci kütüphanesinin ayrıca kendi dil özelinde dokümantasyon seti vardır. Bu destekleyici belge, müşteri seçimi ve ilk yapılandırmada yardımcı olmak için genel bir bakış sağlar.






## QuestDB and C#

### Performans Optimizasyonu İçin En İyi Uygulamalar

- Otomatik Flush Ayarları: Veri gönderimi sırasında otomatik flush ayarlarını yapılandırarak, belirli bir satır sayısına ulaşıldığında veya belirli bir zaman diliminde verilerin gönderilmesini sağlayabilirsiniz. 

- Veri Modelleme: Zaman damgası (timestamp) kullanımı ve semboller (symbols) gibi veri modelleme teknikleriyle sorgu performansını artırabilirsiniz. 

- Sorgu Optimizasyonu: Sorgularınızda LATEST ON gibi QuestDB'ye özgü SQL komutlarını kullanarak, en son verileri hızlı bir şekilde alabilirsiniz


QuestDB doğrudan C# için özel bir resmi client kütüphanesi sunmaz. Ancak bağlantı yolları var. C# tarafında en yaygın PostgreSQL kütüphanesi Npgsql’dir. QuestDB de Postgres protokolünü konuştuğu için doğrudan kullanılabilir. **Adım adım kullanmaya başlayalım**:



#### 1- Docker Desktop bilgisayarınızda varsa açın. Bu sayede bilgisayarınızda Docker'ı çalıştırmış olacaksınız.
#### 2- Komut isteminde bu komutu çalıştırarak Docker'ı başlatın

```bash
docker run -p 9000:9000 -p 8812:8812 -p 9009:9009 questdb/questdb:latest
```


#### Kalıcı kaydetme için
```bash
docker run -d --name questdb -p 9000:9000 -p 8812:8812 -p 9009:9009 -v C:\questdb_data:/var/lib/questdb/db -v C:\QuestDbDemo:/csv questdb/questdb:latest
```


#### 3- Ardından proje dizininize gidin ve paket eklemelerini yapın.

```bash
dotnet add package Npgsql
```

### Program.cs dosyasına bu kodu kopyalayın, QuestDB ile bağlantı sağlarız bu kod ile.

```bash
using Npgsql;

string connString = "Host=localhost;Port=8812;Username=admin;Password=quest;Database=qdb";

using var conn = new NpgsqlConnection(connString);
conn.Open();
Console.WriteLine("QuestDB bağlantısı başarılı!");
```

### CRUD mimarisini karşılayan C# kod örneği:

```bash

using System;
using Npgsql;
using NpgsqlTypes;

class Program
{
    static void Main(string[] args)
    {
        var connString = "Host=127.0.0.1;Port=8812;Username=admin;Password=quest;Database=qdb";

        using var conn = new NpgsqlConnection(connString);
        conn.Open();

        Console.WriteLine("QuestDB bağlantısı başarılı!");

        // Epoch millis
        long tsMillis = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds();

        // INSERT = CREATE
        using (var cmd = new NpgsqlCommand("INSERT INTO my_table (ts, value) VALUES (@ts, @val)", conn))
        {
            cmd.Parameters.Add("@ts", NpgsqlDbType.Bigint).Value = tsMillis;
            cmd.Parameters.Add("@val", NpgsqlDbType.Double).Value = 123.45;

            int rows = cmd.ExecuteNonQuery();
            Console.WriteLine($"{rows} satır eklendi.");
        }


        // DELETE = Truncate ile (Tamamını siler)
        using (var cmd = new NpgsqlCommand("TRUNCATE TABLE my_table", conn)) // Dosyanın tamamını temizleme
        {
            cmd.ExecuteNonQuery();
            Console.WriteLine("Tablo temizlendi.");
        }

        // SELECT örneği (timestamp'i long olarak alıyoruz) = READ
        using (var cmd = new NpgsqlCommand("SELECT cast(ts as long), value FROM my_table LIMIT 5", conn))
        using (var reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                long epochMillis = reader.GetInt64(0);
                DateTime tsValue = DateTimeOffset.FromUnixTimeMilliseconds(epochMillis).UtcDateTime;

                double val = reader.GetDouble(1);

                Console.WriteLine($"ts={tsValue:O}, value={val}");
            }
        }
    }
}

```


### QuestDB’nin daha önce bahsettiğimiz REST API (port 9000) üzerinden sorgu gönderebiliriz. C#’ta bunu `HttpClient` kullanarak yapabiliriz. Örnek:

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var client = new HttpClient();

        string query = "SELECT * FROM weather LIMIT 5";
        string url = "http://localhost:9000/exec?query=" + Uri.EscapeDataString(query);

        var response = await client.GetStringAsync(url);
        Console.WriteLine(response);
    }
}

```
Bu yöntem JSON döndürür, sen de JSON parse ederek C# objelerine dönüştürebilirsin.



### QuestDB'nin, InfluxDB line protocol ile veri kabul ettiğini öğrenmiştik. Bu durumda C#’ta UDP/TCP soketi açıp metin formatında veri gönderebilirsin.(Line Protocol (UDP/TCP ile Veri Yazma) (Örnekteki UDP))

### Örnek: Amacımız C# kullanarak bir sensör verisini UDP üzerinden QuestDB’ye (veya Line Protocol kabul eden bir veritabanına) göndermek.

 ```csharp
using System.Net.Sockets;
using System.Text;

class Program
{
    static void Main()
    {
        using var udpClient = new UdpClient();
        string line = "sensors,location=room1 value=25.3 " + DateTimeOffset.UtcNow.ToUnixTimeMilliseconds() + "000000";
        byte[] data = Encoding.UTF8.GetBytes(line);

        udpClient.Send(data, data.Length, "localhost", 9009);
        Console.WriteLine("Data sent to QuestDB via Line Protocol");
    }
}
```

Burada veriler çok hızlı şekilde QuestDB’ye yazılabilir.


### DOTNET Client

- dotnet add package net-questdb-client

### TCP üzerinden kimlik doğrulama gerekiyorsa, ek olarak şu paketi de kurmanız gerekir:

- dotnet add package net-questdb-client-tcp-auth

### Temel kimlik doğrulama

- using var sender = Sender.New("http::addr=localhost:9000;username=admin;password=quest;"); 

### Token kimlik doğrulama

- using var sender = Sender.New("http::addr=localhost:9000;username=admin;token=<token>");

###  TCP kimlik doğrulama

- using var sender = Sender.New("tcp::addr=localhost:9009;username=admin;token=<token>");



### Temel ekleme (kimlik doğrulaması olmadan)

#### WAL NEDİR? (WAL = Write-Ahead Log)
- Veritabanı teknolojilerinde çok yaygın bir kavramdır.

- Normal tabloda: Veriler doğrudan tabloya yazılır.WAL tabloda: Önce bir log (günlük) dosyasına yazılır, sonra arka planda tabloya işlenir.

**Bu sayede:**

- Yazma çok hızlı olur (çünkü sadece append log yazıyorsun).

- Dayanıklılık artar (çünkü log’dan recovery yapılabilir).

- Concurrent ingestion (çoklu thread/process aynı tabloya aynı anda yazabilir).


#### Avantajları

- Çok yüksek ingestion hızı (milyonlarca row/sn).

- Asenkron commit → yazma hızı artar.

- Crash sonrası recovery daha güvenilir.

#### Dezavantajları

- Biraz daha fazla disk kullanır (çünkü log dosyaları tutulur).

- “İlk yazım” sonrasında veri arka planda işleneceği için, bazı edge case’lerde hemen sorgulanabilir olmayabilir (ama QuestDB bunu hızlı senkronize eder).



#### WALL yapısı ile tablo oluşturuyoruz.

```bash
DROP TABLE trades;

CREATE TABLE trades (
    symbol SYMBOL,
    side SYMBOL,
    price DOUBLE,
    amount DOUBLE,
    ts TIMESTAMP
) TIMESTAMP(ts) PARTITION BY DAY WAL;
```
#### Ardından C# tarafında bu kodu çalıştırarak verileri ekleyebiliyoruz.

```csharp
using System;
using QuestDB;

using var sender =  Sender.New("http::addr=localhost:9000;");
await sender.Table("trades")
    .Symbol("symbol", "ETH-USD")
    .Symbol("side", "sell")
    .Column("price", 2615.54)
    .Column("amount", 0.00044)
    .AtNowAsync();
await sender.Table("trades")
    .Symbol("symbol", "BTC-USD")
    .Symbol("side", "sell")
    .Column("price", 39269.98)
    .Column("amount", 0.001)
    .AtNowAsync();
await sender.SendAsync();

```

#### Zaman damgaları, özel otomatik temizleme, temel kimlik doğrulama ve hata bildirimi içeren bir örnek:

```charp

using QuestDB;
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        using var sender = Sender.New(
          "http::addr=localhost:9000;username=admin;password=quest;auto_flush_rows=100;auto_flush_interval=1000;"
        );

        var now = DateTime.UtcNow;
        try
        {
            await sender.Table("trades")
                        .Symbol("symbol", "ETH-USD")
                        .Symbol("side", "sell")
                        .Column("price", 2615.54)
                        .Column("amount", 0.00044)
                        .AtAsync(now);

            await sender.Table("trades")
                        .Symbol("symbol", "BTC-USD")
                        .Symbol("side", "sell")
                        .Column("price", 39269.98)
                        .Column("amount", 0.001)
                        .AtAsync(now);

            await sender.SendAsync();

            Console.WriteLine("Data flushed successfully.");
        }
        catch (Exception ex)
        {
            Console.Error.WriteLine($"Error: {ex.Message}");
        }
    }
}
```


### Rest API + C# ile belirli bir tarih aralığını getirme ve süresi:

```csharp

using System;
using System.Diagnostics;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        var client = new HttpClient();


        var start = DateTime.UtcNow.AddDays(-30).ToString("yyyy-MM-dd HH:mm:ss");
        var end   = DateTime.UtcNow.ToString("yyyy-MM-dd HH:mm:ss");


        string sql = $"SELECT * FROM veriler WHERE tarih BETWEEN '{start}' AND '{end}' LIMIT 100";

        var url = $"http://10.141.2.7:6587/exec?query={Uri.EscapeDataString(sql)}";

        var sw = Stopwatch.StartNew();
        var response = await client.GetAsync(url);
        sw.Stop();

        Console.WriteLine($"HTTP sorgu süresi: {sw.ElapsedMilliseconds} ms");

        if (response.IsSuccessStatusCode)
        {
            var content = await response.Content.ReadAsStringAsync();
            Console.WriteLine("Sonuç örneği:");
            Console.WriteLine(content);
        }
        else
        {
            Console.WriteLine($"Hata: {response.StatusCode}");
            var errContent = await response.Content.ReadAsStringAsync();
            Console.WriteLine(errContent);
        }
    }
}

```


### QuestDB'de doğrudan `ALTER TABLE RENAME COLUMN` desteği yoktur. Bu nedenle sütun adlarını değiştirmek için tabloyu yeni isimlerle yeniden oluşturmak gerekir.  Aşağıdaki adımlar `test6` tablosu için örneklenmiştir:


```sql

CREATE TABLE test6_new AS (
    SELECT 
        f0 AS id,
        f1 AS device_id,
        f2 AS temperature,
        f3 AS humidity,
        cast(f4 as TIMESTAMP) AS ts
    FROM test6
) TIMESTAMP(ts);



DROP TABLE test6;

RENAME TABLE test6_new TO test6;

SELECT count(*) FROM test6

```

### WAL yapısında nasıl tablo oluşur? "PARTITION BY DAY WAL ekleriz tablo sonuna. Tabloyu oluştururken eklemek zorundayız, yoksa yeni tabloyu WALL ile aşağıdaki şekilde oluşturup mevcut tabloyu geçiririz"

```sql
CREATE TABLE test6_new AS (
    SELECT 
        f0 AS id,
        f1 AS device_id,
        f2 AS temperature,
        f3 AS humidity,
        cast(f4 as TIMESTAMP) AS ts
    FROM test6
) TIMESTAMP(ts) PARTITION BY DAY WAL;
```



**Sırasıyla komutları bu şekilde çalıştırabiliriz. Bu sorgu ile yeni tabloda kayıtların eksiksiz (count ile)taşındığını doğrulayabiliriz.**



### Sorgular genel olarak 5 ms gibi kısa süre almakta.

```sql
SELECT *
FROM test6
WHERE ts BETWEEN '2024-01-28T11:49:00.000Z' AND '2025-02-28T11:55:00.000Z';
```

```sql
SELECT *
FROM test6
WHERE device_id = 1
  AND ts BETWEEN '2025-02-28T11:49:00.000Z' AND '2025-02-28T12:00:00.000Z';
```


### SAMPLE BY ile zaman serisi verilerinde downsampling (örnekleme) yapıyoruz:

```sql
SELECT ts, min(temperature), max(temperature)
FROM test6
SAMPLE BY 1h;
```

```sql
SELECT ts, avg(temperature) AS avg_temp
FROM test6
SAMPLE BY 1m;
```
**Burada her 1 dakikalık pencere için avg_temp hesaplanır.**


### WHERE IN ile birden fazla zaman aralığında arama yaparız.

```sql
SELECT *
FROM test6
WHERE ts IN '2025-02-28T11:49:00.000Z;2025-02-28T12:00:00.000Z',
              '2025-03-01T10:00:00.000Z;2025-03-01T11:00:00.000Z';
```

### LATEST ON ile her cihaz için en güncel kaydı getiririz.

```sql
SELECT *
FROM test6
LATEST ON ts PARTITION BY device_id;
```

### ASOF JOIN ile zaman serisi tablolarını yakın zamana göre joinleriz.

```sql
SELECT a.ts, a.temperature, b.data_id
FROM test6 a
ASOF JOIN veriler b;
```

**Burada test6’daki zaman damgasına en yakın veriler tablosundaki kayıt eşleşir.**



### Materialized Views (Maddeleştirilmiş Görünümler) ile  (Tablo WAL yapısında olmak zorunda. Çünkü materialized view güncel verileri takip edebilmek için WAL özelliğine ihtiyaç duyuyor.)

```sql
CREATE MATERIALIZED VIEW avg_temp_1m AS
SELECT ts, avg(temperature) AS avg_temp
FROM test6
SAMPLE BY 1m;
```
**Örneğin sıcaklıkların dakikalık ortalamasını saklamak için kullanırız.  MV sayesinde sürekli hesap yapmak zorunda kalmayız → sorgular çok daha hızlı çalışır.**



### SAMPLE BY + Fill Politikaları: QuestDB’de eksik verileri doldurma özelliği vardır:

```sql
SELECT ts, avg(temperature)
FROM test6
SAMPLE BY 1m FILL(LINEAR);
```


**Bu, zaman serilerinde eksik veri olduğunda çok faydalı.**

- FILL(NONE) → boş bırakır

- FILL(PREV) → önceki değeri taşır

- FILL(LINEAR) → lineer interpolasyon yapar

- FILL(VALUE x) → sabit değer atar


### Normalde bunu yaparız en güncel verileri almak için fakat aşağıdaki yöntem daha hızlı çalışır.

```sql
SELECT device_id, ts, temperature
FROM (
    SELECT device_id, ts, temperature,
           ROW_NUMBER() OVER (PARTITION BY device_id ORDER BY ts DESC) AS rn
    FROM test6
)
WHERE rn = 1;
```


### LATEST ON ts PARTITION BY key, QuestDB’de zaman serisi tablolarında, her grup için en güncel satırı almak için kullanılır.

```sql
SELECT *
FROM test6
LATEST ON ts PARTITION BY device_id;
```

## ADO.NET + Dapper 


### PGWire portu olup olmadığını anlamak için:


```sql
Test-NetConnection 10.141.2.7 -Port 6588 
```

#### ÇIKTIDA TcpTestSucceeded : True kısmı false ise değildir:

PS C:\Users\hilal> Test-NetConnection 10.141.2.7 -Port 6588


ComputerName     : 10.141.2.7
RemoteAddress    : 10.141.2.7
RemotePort       : 6588
InterfaceAlias   : Wi-Fi
SourceAddress    : 192.168.137.94
TcpTestSucceeded : True



### Gerekli paketler

```sql
dotnet add package Npgsql
dotnet add package Dapper
```

### Kod örneği

```charp

using System;
using System.Threading.Tasks;
using System.Collections.Generic;
using Dapper;
using Npgsql;

namespace QuestDbDemo
{
    class Program
    {
        public class AvgTempRow
        {
            public string Ts { get; set; }
            public double AvgTemp { get; set; }
        }

        public class Measurement
        {
            public string Ts { get; set; }
            public int DeviceId { get; set; }
            public double Temperature { get; set; }
        }

        public class TempRangeRow
        {
            public string Ts { get; set; }
            public double MinTemp { get; set; }
            public double MaxTemp { get; set; }
        }

        static async Task Main(string[] args)
        {
            var connString = "Host=10.141.2.7;Port=6588;Username=admin;Password=quest;Database=qdb;SSL Mode=Disable;Pooling=true;Maximum Pool Size=50;CommandTimeout=30;TimeZone=UTC";
            
            await using var dataSource = NpgsqlDataSource.Create(connString);
            await using var conn = await dataSource.OpenConnectionAsync();

            Console.WriteLine("✅ Bağlantı başarılı!");

            try
            {
                // Mevcut sorgular
                await RunSampleByQuery(conn);
                await RunParameterizedQuery(conn);
                await RunLatestOnQuery(conn);

                Console.WriteLine("\n" + new string('-', 30) + "\n");
                Console.WriteLine("### Yeni Sorgu Örnekleri ###");

                // Yeni eklenen sorgular
                await RunSampleByWithMinMaxQuery(conn);
                await RunLatestOnWithPartitionQuery(conn);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"❌ Hata: {ex.Message}");
                Console.WriteLine($"❌ Stack Trace: {ex.StackTrace}");
            }
        }
        
        /// <summary>
        /// `SAMPLE BY` ile dakikalık ortalama sıcaklık sorgusu.
        /// </summary>
        private static async Task RunSampleByQuery(NpgsqlConnection conn)
        {
            var sqlAvg = "SELECT ts::text AS Ts, avg(temperature) AS AvgTemp FROM test6 SAMPLE BY 1m;";
            var avgRows = await conn.QueryAsync<AvgTempRow>(sqlAvg);
            Console.WriteLine("\n--- Avg Temperature SAMPLE BY 1m ---");
            foreach (var r in avgRows)
            {
                Console.WriteLine($"{r.Ts} -> {r.AvgTemp:F2}");
            }
        }

        /// <summary>
        /// Belirli cihaz ve zaman aralığı için parametreli sorgu.
        /// </summary>
       private static async Task RunParameterizedQuery(NpgsqlConnection conn)
{
    var sqlParams = @"
        SELECT 
            ts::text AS Ts,
            device_id AS DeviceId, 
            temperature AS Temperature
        FROM test6
        WHERE device_id = @DeviceId AND ts BETWEEN @From AND @To
        ORDER BY ts ASC
        LIMIT @Limit;";

    var parameters = new DynamicParameters();
    parameters.Add("@DeviceId", 1);
    parameters.Add("@Limit", 100);
    parameters.Add("@From", DateTime.Parse("2025-02-28T11:49:00Z").ToUniversalTime(), System.Data.DbType.DateTime);
    parameters.Add("@To", DateTime.Parse("2025-02-28T12:00:00Z").ToUniversalTime(), System.Data.DbType.DateTime);

    var measurements = await conn.QueryAsync<Measurement>(sqlParams, parameters);

    Console.WriteLine("\n--- Measurements for device 1 ---");
    foreach (var m in measurements)
    {
        Console.WriteLine($"{m.Ts} | device:{m.DeviceId} | temp:{m.Temperature:F2}");
    }
}

        /// <summary>
        /// Her cihaz için en son ölçümü getiren `LATEST ON` sorgusu.
        /// </summary>
        private static async Task RunLatestOnQuery(NpgsqlConnection conn)
        {
            var sqlLatest = @"
                SELECT 
                    ts::text AS Ts,
                    device_id AS DeviceId, 
                    temperature AS Temperature 
                FROM test6 
                LATEST ON ts 
                PARTITION BY device_id;";
            
            var latest = await conn.QueryAsync<Measurement>(sqlLatest);
            Console.WriteLine("\n--- Latest Measurements per Device ---");
            foreach (var l in latest)
            {
                Console.WriteLine($"{l.Ts} | device:{l.DeviceId} | temp:{l.Temperature:F2}");
            }
        }
        
        // --- Yeni eklenen metotlar ---

        /// <summary>
        /// `SAMPLE BY` ile saatlik en düşük ve en yüksek sıcaklık sorgusu.
        /// </summary>
        private static async Task RunSampleByWithMinMaxQuery(NpgsqlConnection conn)
        {
            var sql = "SELECT ts::text AS Ts, min(temperature) AS MinTemp, max(temperature) AS MaxTemp FROM test6 SAMPLE BY 1h;";
            var tempRows = await conn.QueryAsync<TempRangeRow>(sql);

            Console.WriteLine("\n--- Min and Max Temperature SAMPLE BY 1h ---");
            foreach (var r in tempRows)
            {
                Console.WriteLine($"{r.Ts} -> Min: {r.MinTemp:F2}, Max: {r.MaxTemp:F2}");
            }
        }

        /// <summary>
        /// `LATEST ON` ile her cihaz için en güncel veriyi getiren sorgu.
        /// </summary>
        private static async Task RunLatestOnWithPartitionQuery(NpgsqlConnection conn)
        {
            var sql = @"
                SELECT 
                    ts::text AS Ts,
                    device_id AS DeviceId, 
                    temperature AS Temperature
                FROM test6
                LATEST ON ts PARTITION BY device_id;";

            var latestMeasurements = await conn.QueryAsync<Measurement>(sql);

            Console.WriteLine("\n--- LATEST ON per device_id ---");
            foreach (var m in latestMeasurements)
            {
                Console.WriteLine($"{m.Ts} | device:{m.DeviceId} | temp:{m.Temperature:F2}");
            }
        }
    }
}

```


### SENDER API Kod Örneği:

```charp

using System;
using System.Net.Http;
using System.Threading.Tasks;
using System.Collections.Generic;
using System.Text.Json;

namespace QuestDbHttpDemo
{
    class Program
    {
        public class AvgTempRow
        {
            public string? ts { get; set; }
            public double? avg_temp { get; set; }
        }

        public class Measurement
        {
            public string? ts { get; set; }
            public int? device_id { get; set; }
            public double? temperature { get; set; }
        }

        public class MinMaxTemp
        {
            public string? ts { get; set; }
            public double? min { get; set; }
            public double? max { get; set; }
        }

        static async Task Main(string[] args)
        {
            var baseUrl = "http://10.141.2.7:6587/exec"; 
            using var client = new HttpClient();

            Console.WriteLine("✅ HTTP ile bağlantı hazır");

            try
            {
                await RunCountCheck(client, baseUrl);
                await RunSampleBy(client, baseUrl);
                await RunWhereRanges(client, baseUrl);
                await RunLatestOn(client, baseUrl);
                await RunAsofJoin(client, baseUrl);
                await RunMaterializedView(client, baseUrl);
                await RunFillPolicies(client, baseUrl);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"❌ Hata: {ex.Message}");
                Console.WriteLine($"❌ Stack Trace: {ex.StackTrace}");
            }
        }

        // 1️⃣ COUNT ile doğrulama
        private static async Task RunCountCheck(HttpClient client, string baseUrl)
        {
            string sql = @"
                SELECT count(*) 
                FROM test6
                WHERE ts BETWEEN '2024-01-28T11:49:00.000Z' AND '2025-02-28T11:55:00.000Z';
            ";
            var result = await QueryAsync<Measurement>(client, baseUrl, sql);
            Console.WriteLine("\n--- Count Check ---");
            foreach (var r in result)
                Console.WriteLine($"Count: {r.temperature}");
        }

        // 2️⃣ SAMPLE BY min & max
        private static async Task RunSampleBy(HttpClient client, string baseUrl)
        {
            string sql = "SELECT ts, min(temperature), max(temperature) FROM test6 SAMPLE BY 1h;";
            var result = await QueryAsync<MinMaxTemp>(client, baseUrl, sql);
            Console.WriteLine("\n--- SAMPLE BY 1h (min/max) ---");
            foreach (var r in result)
                Console.WriteLine($"{r.ts} -> min:{r.min} | max:{r.max}");
        }

        // 3️⃣ WHERE IN
        private static async Task RunWhereRanges(HttpClient client, string baseUrl)
{
    // 🔹 Birden fazla zaman aralığını listeye ekliyoruz
    var ranges = new List<(string start, string end)>
    {
        ("2025-02-28T11:49:00.000Z", "2025-02-28T12:00:00.000Z"),
        ("2025-03-01T10:00:00.000Z", "2025-03-01T11:00:00.000Z"),
        ("2025-03-02T08:00:00.000Z", "2025-03-02T09:00:00.000Z") // örnek ek aralık
    };

    // 🔹 SQL WHERE koşulunu dinamik oluşturuyoruz
    var whereConditions = new List<string>();
    foreach (var (start, end) in ranges)
    {
        whereConditions.Add($"(ts BETWEEN '{start}' AND '{end}')");
    }
    string whereClause = string.Join(" OR ", whereConditions);

    string sql = $@"
        SELECT *
        FROM test6
        WHERE {whereClause}
        ORDER BY ts ASC;
    ";

    var result = await QueryAsync<Measurement>(client, baseUrl, sql);
    Console.WriteLine("\n--- Dinamik WHERE RANGES ---");
    foreach (var r in result)
        Console.WriteLine($"{r.ts} | device:{r.device_id} | temp:{r.temperature}");
}


        // 4️⃣ LATEST ON ts PARTITION BY device_id
        private static async Task RunLatestOn(HttpClient client, string baseUrl)
        {
            string sql = "SELECT * FROM test6 LATEST ON ts PARTITION BY device_id;";
            var result = await QueryAsync<Measurement>(client, baseUrl, sql);
            Console.WriteLine("\n--- Latest per Device ---");
            foreach (var r in result)
                Console.WriteLine($"{r.ts} | device:{r.device_id} | temp:{r.temperature}");
        }

        // 5️⃣ ASOF JOIN
        private static async Task RunAsofJoin(HttpClient client, string baseUrl)
        {
            string sql = @"
                SELECT a.ts, a.temperature, b.device_id
                FROM test6 a
                ASOF JOIN test6 b;
            ";
            var result = await QueryAsync<Measurement>(client, baseUrl, sql);
            Console.WriteLine("\n--- ASOF JOIN ---");
            foreach (var r in result)
                Console.WriteLine($"{r.ts} | temp:{r.temperature} | device:{r.device_id}");
        }

        // 6️⃣ MATERIALIZED VIEW (örnek)
        private static async Task RunMaterializedView(HttpClient client, string baseUrl)
        {
            string sql = @"
                CREATE MATERIALIZED VIEW IF NOT EXISTS avg_temp_1m AS
                SELECT ts, avg(temperature) AS avg_temp
                FROM test6 SAMPLE BY 1m;
            ";
            await QueryAsync<AvgTempRow>(client, baseUrl, sql);
            Console.WriteLine("\n✅ Materialized View oluşturuldu (avg_temp_1m)");
        }

        // 7️⃣ SAMPLE BY + FILL Politikaları
        private static async Task RunFillPolicies(HttpClient client, string baseUrl)
        {
            string sql = "SELECT ts, avg(temperature) FROM test6 SAMPLE BY 1m FILL(LINEAR);";
            var result = await QueryAsync<AvgTempRow>(client, baseUrl, sql);
            Console.WriteLine("\n--- SAMPLE BY 1m FILL(LINEAR) ---");
            foreach (var r in result)
                Console.WriteLine($"{r.ts} -> {r.avg_temp}");
        }

        // 🔹 Generic sorgu metodu
        private static async Task<List<T>> QueryAsync<T>(HttpClient client, string baseUrl, string sql)
        {
            var url = baseUrl + "?query=" + Uri.EscapeDataString(sql);
            var response = await client.GetAsync(url);
            response.EnsureSuccessStatusCode();
            var json = await response.Content.ReadAsStringAsync();

            using var doc = JsonDocument.Parse(json);
            var rows = new List<T>();

            if (doc.RootElement.TryGetProperty("dataset", out var dataset))
            {
                var properties = typeof(T).GetProperties();
                foreach (var row in dataset.EnumerateArray())
                {
                    var obj = Activator.CreateInstance<T>();
                    for (int i = 0; i < row.GetArrayLength() && i < properties.Length; i++)
                    {
                        var prop = properties[i];
                        var val = row[i];

                        if (val.ValueKind == JsonValueKind.String)
                        {
                            prop.SetValue(obj, val.GetString());
                        }
                        else if (val.ValueKind == JsonValueKind.Number)
                        {
                            if (prop.PropertyType == typeof(double) || prop.PropertyType == typeof(double?))
                                prop.SetValue(obj, val.GetDouble());
                            else if (prop.PropertyType == typeof(int) || prop.PropertyType == typeof(int?))
                            {
                                if (val.ValueKind == JsonValueKind.Number)
                                    prop.SetValue(obj, val.GetInt32());
                                else if (val.ValueKind == JsonValueKind.String && int.TryParse(val.GetString(), out int intVal))
                                    prop.SetValue(obj, intVal);
                            }

                        }
                    }
                    rows.Add(obj);
                }
            }
            return rows;
        }
    }
}


```

**Bu kodda veriler fazla olduğu için kod hatalı çalışabilir.**


## QuestDB .NET Kullanım Yöntemleri

| Yöntem             | Özellikler                                                                 |
|--------------------|------------------------------------------------------------------------------|
| **Sender API**     | - Sadece **WAL tabloları** destekler <br> - Çok hızlı, **real-time ingestion** için tasarlanmış |
| **ADO.NET + Dapper** | - Hem **WAL** hem **non-WAL** tablolarıyla çalışır <br> - Normal **SQL** yazabilirsin (`SELECT`, `INSERT`, `UPDATE`, `JOIN` vs.) <br> - ORM kolaylığı sağlar |


## QuestDB .NET Kullanım Yöntemleri Karşılaştırma

| İşlem Türü                          | En Uygun Yöntem     | Açıklama |
|-------------------------------------|---------------------|----------|
| **INSERT (çok hızlı, akış)**        |  Sender API        | Line Protocol ile çok yüksek throughput sağlar. Özellikle IoT, sensör, log gibi sürekli veri akışlarında idealdir. |
| **INSERT (az miktar, normal)**      |  ADO.NET + Dapper  | Küçük hacimli veri eklemeleri için uygundur. Kullanımı kolaydır ama Sender kadar hızlı değildir. |
| **SELECT (arama, filtre, BETWEEN)** |  ADO.NET + Dapper  | PostgreSQL wire protocol üzerinden tam SQL sorgu desteği sunar. Aralık sorguları için en hızlı ve güvenilir yöntemdir. |
| **Analitik sorgular (GROUP BY, JOIN, ORDER)** |  ADO.NET + Dapper | SQL tam desteği olduğu için karmaşık analizlerde tercih edilir. |
| **Gerçek zamanlı stream yazma**     |  Sender API        | Gerçek zamanlı veri ingestion için en yüksek performansı sağlar. |

---

### Öneri
- **Veri yazma (INSERT)** → **Sender API** (yüksek hacim için).  
- **Veri okuma (SELECT, aralık sorgusu, analiz)** → **ADO.NET + Dapper**.  

(QuestDB'ye import etme) Yükleme yöntemleri: COPY FROM, REST API, Influx Line Protocol, PostgreSQL wire protokolü, Kafka/MQTT entegrasyonu.


# 📚 Kaynakça

- [QuestDB Demo](https://demo.questdb.io/index.html) – Online demo ortamı  
- [QuestDB Download](https://questdb.com/download/) – QuestDB indirme sayfası  
- [QuestDB GitHub](https://github.com/questdb) – Resmî GitHub adresi  

## 🔹 Canlı Dashboard Örnekleri
- [Crypto Dashboard](https://questdb.com/dashboards/crypto/) – Canlı kripto verileri  
- [FX Orderbook Dashboard](https://questdb.com/dashboards/fx-orderbook/) – Canlı FX Orderbook verileri  

## 🔹 Grafana Entegrasyonu
- [Grafana Plugin (QuestDB Datasource)](https://grafana.com/grafana/plugins/questdb-questdb-datasource/)  
- [Grafana Kullanımı Dokümantasyon](https://questdb.com/docs/third-party-tools/grafana/)  
- [QuestDB Demo (Görselleştirme)](https://demo.questdb.io/index.html)  

## 🔹 SQL & Veri Yapıları
- [QuestDB Veri Tipleri](https://questdb.com/docs/reference/sql/datatypes/)  
- [SQL Overview](https://questdb.com/docs/reference/sql/overview/#postgresql)  

## 🔹 Ingestion & API
- [ILP Protocol Overview](https://questdb.com/docs/reference/api/ilp/overview/)  

## 🔹 .NET / C# Kullanımı
- [QuestDB .NET Client](https://questdb.com/docs/clients/ingest-dotnet/)  
- [Medium Yazısı: QuestDB ile Zaman Yolculuğu (C#)](https://medium.com/@sumerburak/questdb-ile-zaman-yolculu%C4%9Funa-%C3%A7%C4%B1k%C4%B1yoruz-net-ile-e%C4%9Flenceli-bir-macera-c7bea81d4af1)  
