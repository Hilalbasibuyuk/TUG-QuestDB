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

Varsayılan veritabanının adı'dır qdbve bu, yapılandırma aracılığıyla değiştirilebilir. Ancak, standart bir SQL veritabanının aksine, USE DATABASEkomut vermenize gerek yoktur. Bağlandıktan sonra, hemen sorgulamaya ve veri eklemeye başlayabilirsiniz.


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



**GRAFANA**
Grafana, verileri görselleştirmek ve zaman serisi veri analizini etkinleştirmek için kullanılan popüler bir gözlemlenebilirlik ve izleme uygulamasıdır. 


## Rest API
QuestDB REST API, standart HTTP özelliklerine dayanır ve hazır HTTP istemcileri tarafından anlaşılır. QuestDB ile etkileşim kurmanın basit bir yolunu sunar ve çoğu programlama diliyle uyumludur. API işlevleri URL'ye tam olarak bağlıdır ve argüman olarak sorgu parametrelerini kullanır.

**Mevcut yöntemler**
### /imp (Import data) for importing data from .CSV files.
Tablo biçimindeki metin verilerini doğrudan bir tabloya aktarır. İsteğe bağlı başlıklarla CSV, TAB ve boru ( ) ile ayrılmış girdileri destekler . Veri boyutu konusunda herhangi bir kısıtlama yoktur. Veri türleri ve yapıları, ek yapılandırmaya gerek kalmadan otomatik olarak algılanır.

**Örnek kullanım:**
```bash
curl -F data=@weather.csv \
'http://localhost:9000/imp?overwrite=true&name=new_table&timestamp=ts&partitionBy=MONTH'

# Standart dışı zaman damgasına sahip CSV'yi içe aktarma
curl -F data=@weather.csv 'http://localhost:9000/imp'

#User defined schema
curl \
-F schema='[{"name":"dewpF", "type": "STRING"}]' \
-F data=@weather.csv 'http://localhost:9000/imp'
```


### /exec to execute a SQL statement.
SQL INSERT Query, Giriş noktası bir SQL sorgusu alır ve sonuçları JSON olarak döndürür. Bunu hızlı SQL eklemeleri için de kullanabiliriz, ancak SQL enjeksiyonu sorunlarından kaçınmak için gerekli olan parametreli sorgular için destek olmadığını unutmayın. Yüksek performanslı eklemelere ihtiyacınız varsa **InfluxDB Satır Protokolünü** tercih edin.

**Örnek kullanım:**


```bash
# Tablo Oluşturma
curl -G \
  --data-urlencode "query=CREATE TABLE IF NOT EXISTS trades(name STRING, value INT)" \
  http://localhost:9000/exec

# Satır Ekleme
curl -G \
  --data-urlencode "query=INSERT INTO trades VALUES('abc', 123456)" \
  http://localhost:9000/exec
```

  
### /exp to Export Data.
Bu uç nokta, URL kodlu sorguları geçirmenize olanak tanır ancak istek gövdesi, JSON'un aksine kaydedilip yeniden kullanılmak üzere tablo biçiminde döndürülür. Veritabanını bir SQL select sorgusu ile sorgulamaya ve sonuçları CSV olarak almaya olanak tanır. Sonuçları JSON formatında almak için /exec kullanırız.

```bash
# Sorguyu göz önünde bulundurarak:

curl -G \
  --data-urlencode "query=SELECT AccidentIndex2, Date, Time FROM 'Accidents0514.csv'" \
  --data-urlencode "limit=5" \
  http://localhost:9000/exp
```


### Authentication(RBAC(Role-based Access Control))

#### REST API supports two authentication types:
- HTTP basic authentication
- Token-based authentication

İlk kimlik doğrulama türü çoğunlukla web tarayıcıları tarafından desteklenir. Ancak, kullanıcı kimlik bilgilerini bir başlıkta programatik olarak da uygulayabilirsiniz.(Authorization: Basic)

```bash
curl -G --data-urlencode "query=SELECT 1;" \
    -u "my_user:my_password" \
    http://localhost:9000/exec
```

İkinci kimlik doğrulama türü, bir REST API belirtecinin bir başlıkta belirtilmesini gerektirir  (Authorization: Bearer header)
```bash
curl -G --data-urlencode "query=SELECT 1;" \
    -H "Authorization: Bearer qt1cNK6s2t79f76GmTBN9k7XTWm5wwOtF7C0UBxiHGPn44" \
    http://localhost:9000/exec
```

## PostgreSQL kablolu protokol istemcileri
### PostgreSQL ve PGWire

QuestDB, veri girişi için Postgres Wire Protokolünü (PGWire) destekler. **C# ile kullanacağımız zamanlarda da PGWire ile bağlanacağız**. PGWire (Postgres Wire Protokolü) PostgreSQL'in istemci-sunucu iletişim protokolüdür. Sorgulama ve veri çıkışı için QuestDB, PostgreSQL protokolüyle uyumludur. Bu, QuestDB ile favori PostgreSQL istemcinizi veya sürücünüzü kullanabileceğiniz anlamına gelir. PGWire arayüzü, öncelikle QuestDB'den veri sorgulamak için önerilir. Veri alımı, özellikle yüksek verimli senaryolar için QuestDB, InfluxDB Hat Protokolü'nü (ILP) destekleyen istemcilerinin kullanılmasını önerir . Bu istemciler, hızlı veri girişi için optimize edilmiştir. 

**Not:** PostgreSQL depolama modeli QuestDB'den temel olarak farklıdır. Sonuç olarak PostgreSQL için var olan bazı özellikler QuestDB'de bulunmamaktadır.


### PGWire İstemcisine Genel Bakış:
QuestDB, istemcilerin PostgreSQL istemci kitaplıklarını kullanarak QuestDB'ye bağlanmasını sağlamak için PostgreSQL kablolu protokolünü (PGWire) kullanır. Bu, mevcut PostgreSQL istemcilerini ve kitaplıklarını kullanmanıza olanak tanıdığı için QuestDB'yi kullanmaya başlamak için harika bir yoldur. 

#### Sorgulama ve veri arayüzü
PGWire arayüzü, öncelikle QuestDB'den veri sorgulamak için önerilir. Veri alımı, özellikle yüksek verimli senaryolar için QuestDB, InfluxDB Hat Protokolü'nü (ILP) destekleyen istemcilerinin kullanılmasını önerir . Bu istemciler, hızlı veri girişi için optimize edilmiştir.


#### Zaman dalgası işleme
QuestDB, tüm zaman damgalarını dahili olarak UTC(Coordinated Universal Time) olarak depolar . Ancak, zaman damgalarını PGWire protokolü üzerinden iletirken QuestDB bunları "TIMESTAMP WITHOUT TIMEZONE" olarak gösterir. Bu durum, istemci kütüphanelerinin bu zaman damgalarını varsayılan olarak kendi yerel saat dilimlerinde yorumlamasına ve bu da olası karışıklığa veya hatalı veri gösterimine yol açabilir. Dil özelindeki kılavuzlarımız, istemcinizin bu zaman damgalarını UTC olarak doğru şekilde yorumlayacak şekilde nasıl yapılandırılacağına dair ayrıntılı örnekler sunar.

Mevcut davranış şu an iyileştirilmeye çalışılıyor. Bu arada, zaman damgalarının tutarlı bir şekilde işlenmesini sağlamak için istemci kitaplığınızdaki saat dilimini UTC olarak ayarlamanızı öneririz.

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



## ILP Protokolü ile Şema otomatik oluşturma
Influx Line Protocol (ILP) kullanıldığında , QuestDB gelen verilere göre otomatik olarak tablolar ve sütunlar oluşturur.Bu özellik, InfluxDB'den geçiş yapan veya InfluxDB istemci kütüphaneleri ya da Telegraf gibi araçlar kullanan kullanıcılar için kullanışlıdır , çünkü önceden tanımlanmış şemalar olmadan verileri doğrudan QuestDB'ye gönderebilirler. Ancak bunun bazı sınırlamaları vardır:
- QuestDB, otomatik olarak oluşturulan tablolara ve sütunlara varsayılan ayarları uygular (örneğin, bölümlendirme, sembol kapasitesi ve veri türleri).
- Bölümlemeyi veya sembol kapasitesini daha sonra değiştiremezsiniz .
- Veri türünü otomatik olarak oluşturamazsınız IPv4. Bir IP adresini dize olarak göndermek bir sütun oluşturacaktır VARCHAR.
Sütun otomatik oluşturmayı yapılandırma yoluyla devre dışı bırakabilirsiniz . QuestDB, verileri almak için InfluxDB Hat Protokolünü uygular. InfluxDB Hat Protokolü yalnızca veri alımı içindir Her ILP istemci kütüphanesinin ayrıca kendi dil özelinde dokümantasyon seti vardır. Bu destekleyici belge, müşteri seçimi ve ilk yapılandırmada yardımcı olmak için genel bir bakış sağlar.








## QuestDB and C#
QuestDB doğrudan C# için özel bir resmi client kütüphanesi sunmaz. Ancak bağlantı yolları var. C# tarafında en yaygın PostgreSQL kütüphanesi Npgsql’dir. QuestDB de Postgres protokolünü konuştuğu için doğrudan kullanılabilir. Adım adım kullanmaya başlayalım:

```bash
# NuGet üzerinden kütüphane ekle

dotnet add package Npgsql
```

```bash
# Bağlantı aç ve sorgu gönder

using System;
using Npgsql;

class Program
{
    static void Main()
    {
        // QuestDB, default olarak port 8812’de PostgreSQL protokolünü dinler
        var connString = "Host=localhost;Port=8812;Username=admin;Password=quest;Database=qdb";

        using var conn = new NpgsqlConnection(connString);
        conn.Open();

        // Tablo oluştur
        using (var cmd = new NpgsqlCommand("CREATE TABLE IF NOT EXISTS sensors(ts TIMESTAMP, value DOUBLE)", conn))
        {
            cmd.ExecuteNonQuery();
        }

        // Veri ekle
        using (var cmd = new NpgsqlCommand("INSERT INTO sensors VALUES (now(), 42.5)", conn))
        {
            cmd.ExecuteNonQuery();
        }

        // Veri oku
        using (var cmd = new NpgsqlCommand("SELECT * FROM sensors", conn))
        using (var reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine($"{reader.GetDateTime(0)} => {reader.GetDouble(1)}");
            }
        }
    }
}

// Bu yöntemle, QuestDB’ye PostgreSQL gibi bağlanabiliyoruz
```


QuestDB’nin daha önce de bahsettiğimiz REST API’sini (port 9000) hatırlayalım.
C#’ta HttpClient kullanarak sorgu gönderebilirsin.


bash ```
# Rest API kullanma

using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var client = new HttpClient();
        
        string query = "SELECT * FROM sensors LIMIT 5";
        string url = $"http://localhost:9000/exec?query={Uri.EscapeDataString(query)}";

        var response = await client.GetStringAsync(url);
        Console.WriteLine(response);
    }
}

# Bu yöntem JSON döndürür, sen de JSON parse ederek C# objelerine dönüştürebilirsin.

```



QuestDB'nin, InfluxDB line protocol ile veri kabul ettiğini öğrenmiştik. Bu durumda C#’ta UDP/TCP soketi açıp metin formatında veri gönderebilirsin.


bash ```
# Line Protocol (UDP/TCP ile Veri Yazma) (Örnekteki UDP)

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

# Burada veriler çok hızlı şekilde QuestDB’ye yazılabilir.
```


En sağlam yöntem → ADO.NET + Dapper.

Grafana konusuna bakıcam(Docker ile çalışıyor)



# KAYNAKÇA
- https://demo.questdb.io/index.html    -> Demo
- https://questdb.com/download/         -> Download QuestDB
- https://github.com/questdb             -> QuestDB GitHub address
- https://questdb.com/dashboards/crypto/    -> Canlı crypto verileri
- https://questdb.com/dashboards/fx-orderbook/    -> Canlı fx-orderbook verileri
- https://grafana.com/grafana/plugins/questdb-questdb-datasource/    -> Grafana kullanımı
- https://questdb.com/docs/third-party-tools/grafana/   -> Grafana kullanımı
- https://demo.questdb.io/index.html   -> Görselleştirme yapılabilir bu yapı kullanılarak
- https://questdb.com/docs/reference/sql/datatypes/   -> QuestDB veri yapıları listesi
- https://questdb.com/docs/reference/sql/overview/#postgresql  -> Query & SQL Overview (SQL sorguları)
- https://questdb.com/docs/reference/api/ilp/overview/    -> ILP protokolü detayları. kullanıcam.
- https://questdb.com/docs/clients/ingest-dotnet/     -> C# ile QuestDB (Yarın buna bakıcam, c# ile yapmamız gereken şeyleri hocaya sorucam, ACID,CRUD mimairisyle ilerilcem, dotnet )




