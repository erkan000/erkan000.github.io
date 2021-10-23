---
layout: default
title: EDA, Kafka, Ksql
date: 23.10.2021
---

## EDA - Event Driven Architecture

A software architecture pattern promoting the production, detection, consumption of, and reaction to events. İletişimi data ile değil, eventler ile yapıldığı model diyebiliriz. Şu kısımlardan oluşur.
- Microservices
- Serverless
- FaaS (Function as a service)
- Streaming Model (Real time responsiveness)
- Event Sourcing(Eventlerin sıralı olarak tutulması ve istendiğinde durumların tekrar oluşturulabilmesidir)
- CQRS (Command Query Responsibility Segregation, okuma ve yazmanın farklı nesneler ile yapılmasıdır.)

### EDA faydaları
- Decoupling, bütün bileşenler decoupled dır. İletişimimizi brokerları kullanarak yaparız, böylece servisler birbirlerini bilmezler, herşeyi broker yönetir. Servisler birbirine bağımlı olmamış olur.
- Encapsulation, bütün eventler belirli bir olayı temsil eder ve aralarındaki sınır belirlidir. Her eventin kendi sorumluluğu vardır.
- Speed, eventler real-time işlenir.
- Scalability

### EDA Dezavantajları
- Source of truth
- Duplicate events
- Öğrenme eğrisi
- Complex
- Loss of transactionality
- Lineage

### Event Storming
- Typical system               -   Revolves around data
- Event-driven system     -   Resolves around event
EDA da datanın ne olduğu önemli değildir, eventlerin nasıl işlendiği önemlidir. Yeni bir projeye başlarken event stroming denen bir workshop yapılır ve DDD kullanılarak gereksinimler belirlenir. Event Storming bütün iş modellerini domain eventler ile birlikte modellenmesini sağlar.



## Kafka
Kafka özellikleri (Zero copy concept)
- Open source 
- Skala ile yazılmaya başlandı, sonra Java ile yazıldı
- High throughput
- More than a messaging system, is a distributed streaming platform.

### Distributed streaming platform
- Messaging system, produce and consume messages
- Distributed storage system, data stores in fault tolerant way, so prevent data loss
- Data processing, real-time data processiing

Kafka binary veri ile çalışır. Serialization veya deserialization yapmaz. Bütün veri binary olarak networkden gelir ve binary olarak disk e yazılır. (TCP)

### Kafka architecture
2 ana yapıdan oluşur, broker ve zookeper.

- Broker, veriyi alır, gönderir ve saklar. Donanım üstünde koşan bir yazılımdır, yüksek hız için birden fazla tanımlanır.
- Zookeeper, brokerları yönetir. configleri brokerlara verir, bozulan brokera veri yollamayı engeller. Lider brokerı seçer ve diğerlerine anons eder.
Yukardaki yapıya kafka cluster denir. Genelde bu uygulamalar farklı sunucularda koşar.

### Kurulum
- Önce zookeeper çalıştırılır. config ayarından clientPort:2181 değiştirilerek istenen portu dinlemesi sağlanabilir. 
- ardından istediğimiz kadar broker ayarlayayıp çalıştırabiliriz. herbirine şu ayarları yaparız
   - broker.id=1 brokerı tanımlar, hepsi uniq olmalı
   - listeners=PLAINTEXT://9093 , broker dinleyeceği port ve protokolü(şifreli değil, plain text)
   - zookeeper.connect=localhost:2181 , zookeperın çalıştığı host ve portu belileriz.

### Records
Kafka ya gelen ve giden mesajlara "Record" denir. 3 kısımdan oluşur. Key, value, timestamp. Key ve value herhangi birşey olabilir, timestamp ise bir sayıdır, timestamp formatında. Mesajın üretildiği zamanı belirtir.

### Topics
Mesajların gruplandırılmasıdır. Broker içinde tutulurlar. Hangi topic'in hangi brokerda olduğu bilgisi ise zookeeper da tutulur. 2 tür topic vardır, delete ve compact.
- Delete topic, topic size'ı büyüyünce eski mesajları siler veya belirli bir süreden eski mesajları siler. default 7 gün. Bu süreye "retention time" denir. 
- Compacting topics, UPSERT mantığı ile çalışır. Update, insert. Her gelen mesajın key'ine bakar, aynı key topicde varsa value sunu update eder, yoksa key/value değerini ekler. 

Kafka yükü birden fazla broker'a dağıtır. Bunu partition lar ile yapar. Oluşturulan bir topic farklı partitionlara bölünür ve her partition da farklı brokerlarda saklanır. Burada her partition farklı veri saklarlar. Peki yedeklilik nası sağlanır? Aslında arka planda her brokerın diğer partition'ı diğer broker'ın partition ına otomatik olarak yedeklenir. Böylece bir broker down olunca, diğer brokerda zaten down olan brokerın yedek mesajları bulunmaktadır.

### Producers
Kafka ya veri göndermek için gerekli bütün işi yapan sınıflardır. Bir nesneyi alırlar, key ve value değerlerini binary formata çevirirler ve network den gönderirler. 

### Topic yaratma
- topic ismi belirlenir
- topigin hangi brokerda oluşturulacağı belilenir, localhost:9083 gibi
- kaç partition olacağı belirlenir, mesela 2(partitions)
- kaç kopyası olacağı belirlenir, mesela 2(replication-factor)
Yukarıdaki tanımla aslında 4 partition oluşturduk, 2 si yedek, 2 si asıl olmak üzere.

### Mesaj yollamak
Producer yaratmak için şu değerler kafidir
- bootstrap.servers
- key.serializer
- value.serializer
kafka-clients kütüphanesi ile Producer yukardaki properties ile oluşturulur. producer.send metodu ile gönderebiliriz.

### Consumers
Consumer lar pull mantığı ile çalışır, kafka mesajları push etmek, consumerlar çeker. Böylece yavaş consumerlar kafkayı yavaşlatmazlar. Consumer başlayınca, Belirli bir periyotta kafkaya benim için mesaj varmı diye sürekli sorar. Mesaj gelince yine binary formattadır. Bu veriyi deserialize eder.

### Mesaj almak
Consumer yaratmak için şu değerler kafidir
- bootstrap.servers
- group.id
- key.deserializer
- value.deserializer
kafka-clients kütüphanesi ile Consumer yukardaki properties ile oluşturulur. consumer.subscribe metodu topic e abone oluruz. consumer.pool(Duration) ile ise consumer'ı belirtilen Duration kadar dinleriz. Sonra gelen bütün mesajları alabiliriz. 

## Communication with AVRO and message registry
Bir nesneyi serialize ederek network ile başka sunucuya, veya diske yazabilir, bu veriyi daha sonra deserialize edebiliriz. Burada serialize ettikten sonra binary olarak veya plaintext olarak gönderebiliriz. Binary hızlıdır fakat debug etmesi zordur, okunamaz. 
Diğer bir husus ise serialization formatın veri yapısını doğrulayıp doğrulamadığıdır. Bu Schema lar vasıyasıyla validate edilebilir. Mesela protobuff, XML, JSON vs..

AVRO data serialization sistemidir. Veriyi binary olarak iletir ve şema validasyonu yapar. Binary olduğu için JSON ve XML e göre daha hızlıdır. Şema ile beraber serDe yapılabilir. Tipler diğer notumda var.

### Schema Registry
Producer ve consumer lar için şema dağıtımı uygulamasıdır. Şemalarını bir kafka topic içine yazar. Uygulama şu şekilde çalışır. Producer kafkaya nesne göndermek için SR ye bu sınıfın şemasını bana ver, der. SR şemayı, ona özel bir şema ID ile gönderir. Producer bu şema ile mesajı serialize edip şema ID ile beraber kafkaya yollar. Kafkayı dinleyen consumer mesajı alır ve deserialize etmek için önce SR ye bana şu şema ID li şemayı ver der. SR ilgili şemayı consumer a yollar ve consumer da ilgili şema ile veriyi deserialize edebilir. Görüldüğü üzere servisler birbiri ile decoupled!

### Demo
SR yi confluent github dan indirebilir(https://github.com/confluentinc/schema-registry) ve mvn package ile derleyebiliriz. Ardından default ayar dosyası ile başlattığımızda 8081. porttan dinlemeye başlar. Yukarıdaki producer ayarlarından farklı olarak

- key.serializer ve value.serializer değerleri "KafkaAvroSerializer" olarak değiştirilir.(kafka-avro-serializer bağımlılığında)
- "schema.registry.url" olarak localhost:8081 girilir.

Consumer da ise yukardaki değişikliklere ek olarak "specific.avro.reader" true yapılır ve tipler arası otomatik dönüşümün yapılması sağlanır.

## Kafka Streams
Stream veri biliminde data parçalarının tek tek işlenmesidir. Mesela bir fraud uygulaması yapabiliriz. UI gelen ödemeleri sürekli payments topic'ine yazar. Fraud stream uygulamamız ise buraya gelen mesajları sürekli dinler ve bazı kuralları geçer ise bunu validated-payments isimli bir topic'e yazabilir. (Dispatch) Onu da başka bir processor dinleyebilir. 

Stream uygulaması aslında bir topic consumer ve producer dan ibarettir. Streams bize bu yapıyı hazır sunar. Böylece stream uygulamasının business kısmına odaklanabiliriz. Stream kısmı ise mesajı çeşitli aşamalardan geçirerek işler. Bu aşamalara "topology" denir. "Topolgy is acyclic(zincir biçiminde olan) graph of sources, processors, and sinks. 

- Source, Consumer tipinde özel bir processordur. Stream uygulamasına göre input temsil eder. 
- Sink, Producer tipinde özel bir processordur. Stream uygulamasına göre output temsil eder. 
- Stream uygulamasında en az bir tane Source Processor olmalıdır!
- Stream uygulaması mesajları tek tek işler. Stream chain de işlenip output topic'e yolladıktan sonra diğer mesajı consume eder.
- Her bir processor belirli bir iş yapıp, stream chain deki diğer processor'a verir.
- Filter filtreleme yapar, map veriyi dönüştürür, count ise mesaj sayar ve adet döndürür.
- count processor kendine gelen mesajları belirli bir tipe göre sayabilir. Streamde her mesaj tek tek işlendiği için bunu saklayacak bir yere ihtiyaç duyar. Buna "State Store" denir. İki çeşit State Store var
   - Ephemeral, uygulama çökünce veri kaybolur.
   - Fault-tolerant, veriyi bir yere kaydeder.(Default'u budur ve bir topice kaydeder. )

### Duality of Streams 
Streams uygulamaları sadece stream kullanmak için yapılmaz. Veriyi veritabanında da saklamamız gerekir. Çünkü stream girişi olan ve stream çıkışı olan bir processor, diğer eventlerin birbirleri arasında bir bağlantı yoktur. Hepsini birbirinden bağımsız işler. Bu stream topic eventleri genellikle belirli süre sonra silinir. (Delete topics lerde saklanırlar.) Fakat banka hesabı gibi birşeyde hesap hareketlerinin hepsi banka hesabını oluşturur ve her işlemin diğer işlemlerden haberdar olması gerekir. 

Çözüm için tablolar geliştirilmiştir. Bir streamde gerçekleşen son olaylar, tablo olarak saklanabilir. (Compaction topics lerde saklanırlar.) 

Stream türünde bir akış, tabloya çevrilebilir veya tablo bir stream'e çevrilebilir. Stream'i tabloya çevirmek için aggregate(), reduce(), count() gibi metodlar kullanılır. Tabloyu stream'e çevirmek için ise sadece tablo kayıtlarında for ile gezeriz, daha kolayı toStream() metodunu kullanırız.

### Processors
- Stateless processors, işlemek için duruma ihtiyaç duymaz, durum bağımsızdır.
- Statefull processors, işlem yapmak için state store'a ihtiyaç duyar.

### Stateless Operations

Java 8 streams API çok benziyor.

- branch, bir streami birden fazla stream'e ayırmak için
- filter, istenmeyen mesajları reject eder
- inverse filter, filter'ın tersi
- map, mesaj tipini dönüştürmek için
- flatMap, mesajı birden fazla mesaja dönüştürmek için
- foreach, kullanılınca stream terminate olur, tekrar kullanamzsın!!!
- peek, foreach gibi stream'i consume etmeden foreach yapar.
- groupBy, mesaj key veya value nun bir alanına göre gruplar
- merge, iki streami birleştirir.

### Stateful Operations
- aggregations, mesela bir topice gelen bütün mesajların toplamını hesaplar
- count, counts messages with same key
- join, mesajı başka bir topicdeki bilgilerle zenginleştir
- windowing, belirli sürede yapılacakları hesaplar, key lere bakar.
- custom processors

## KSQL
Ksql, kafka streams API'nin üstüne kurulmuş, stream uygulamaları yazmamızı sağlayan bir API dir. Confluent tarafından yazılmış ve open source dur. Amacı streams uygulamaları yazmamızı kolaylaştırmaktır. 

### Çalışma mantığı
KSQL server kurarız, içinde streams uygulamalarımız çalışacaktır. Bu server'ın kafka cluster'ımıza bağlanmasını sağlarız. Çünkü cluster da topicleri produce ve consume edecektir. Peki bu serverda stream uygulamaları nasıl oluşur? Kullanıcı KSQL yazarak aslında streams uygulamaları yazmış olur. Kullanıcı bu KSQL cümleleri development ortamında "ksql-cli" üzerinde yazıp kafka server'a iletir. Bu iletim REST ile yapılır.

Yazılan her KSQL sorgusu bir stream uygulaması oluşmasına sebebiyet verir. Sorgu sonucu serverda bir topoloji oluşur ve otomatik olarak çalışmaya başlar.

Kullanma sebebimiz ise şunlar olabilir. Öncelikle çok basit bir select ile stream uygulaması oluşturabiliriz. Topic lerdeki verileri okumamızı çok kolaylaştırır. Ayrıca datayı manupile etmemizi de çok kolaylaştırır.

### Windowing
Belirli zaman aralığında gelen mesajları sayar. Mesajları key değerine göre gruplayarak sayar. 2 çeşit yapılır.
- Tumbling, frame sabittir, 0-30, 30-60. dakika gibi
- Hopping, frame hop interval'e göre kayar. Diyelim hop interval 10 ise, zaman 0-30, 10-40, 20-50, 30-60 olarak ileri doğru kayar. Ortak mesajlar tekrar tekrar sayılır.

### KSQL Statements
- DDL
- DML



CREATE STREAM AS SELECT, 2 iş yapar
- Bir stream oluşturur, default delete olarak
- Select in sonucu oluşturduğu verileri bu topic e yazar.

### KSQL kurulum
- github dan ksql çekilir(https://github.com/confluentinc/ksql)
- git checkout v5.2.0 brancina geçilir.
- mvn package 

### KQL ayarları
- config/ksql-server.properties dosyası edit edilir.
- bootstrap.servers değeri ile cluster set edilir.
- bin/ksql-server-start config/ksql-server.properties
- default olarak localhost:8088 i dinler.

### KSQL CLI çalıştır
- bin/ksql (default localhost:8088 e bağlanır. İstenirse farklı cihazlarda da olabilir  server ve cli)

### CLI komutları
- SHOW STREAMS;
- SHOW TOPICS;
- CREATE STREAM;
- CREATE TABLE;
- PRINT 'topicName'
- PRINT 'tableName'

CREATE STREAM ksql_payments
WITH ( KAFKA_TOPIC='payments', VALUE_FORMAT='AVRO');

Yukarıdaki komutla yeni bir stream yaratılır. payments topicini izler ve payments topici value formatı AVRO dur.

Aşağıda ise son 10 dakikada kullanıcıların gerçekleştirdiği transaction sayılarını görmek istedik. Bu işi stream ile yapamayız o yüzden table oluşturuyoruz.

CREATE TABLE warnings
AS SELECT userId, COUNT(*)    // value formatı burada belirlenir
FROM ksql_payments
WINDOW HOPPING(SIZE 10 MINUTES, ADVANCE BY 1 MINUTE)
GROUP BY userId     // count kullandığımız için gruplamamız lazım
HAVING COUNT(*) > 5;


## Kafka Connect
Kafka Connect, bir kafka cluster dan dış sistemlere hiçbir kod yazmadan topiclere gelen verileri dışarı yazmaya yarar. Destekledikleri;
- MongoDB
- Neo4J
- ElasticSearch
- Solr
- AWS S3
- Twitter

Kafka connect bunu yaparken her mesajı ilgili servise tek tek yollamaz! Önce kendi internal memory sinde tutar ve belirli aralıklarla dış servise yazar. Amaç gereksiz overhead i önlemek, transaction sayısını azaltmak.

Kafka connect birden fazla oluştururlarak scale edilebilir. Development için standalone kullanılır, prod için distributed olanı kullanılır.

### Connectors,
Kafka verilerini persist etmek için oluşturulan hazır API lerdir. 
- Source connector, kaynaktan kafkaya iletmek için
- Sink connector, kafkadan dış sisteme iletmek için

DB den kafkaya iletmek için, JDBC source connector kullanılır.
Kafkadan AWS e iletmek için, AWS S3 Sink connector kullanılır.



## Kafka REST
Kafka iletişimini binary yapar demiştik. Bu iletişim OSI layer 4 de, TCP protokolünde olur. Bu sebepten şifreli değildir veriler. Şifrelemek istersek layer 6 ya çıkmamız ve SSL kullanmamız gerekir. Dezantajı ise hız azalır, overhead artar. 

### Kafka iletişim mimarisi
Producer bir mesajı TCP olarak ACK yapmadan direk olarak kafkaya gönderir. Kafka ise sadece mesajı aldım diye ACK yollar. Bu iletişim aynı zamanda asenkron gerçekleşir. 

Eğer uygumamamıza kafka-clients ekleyemezsek, kafkanın önüne REST proxy ekleriz. Bu proxy kafkadan mesaj produce eder veya consume eder. Legacy uygulamamız ise GET ve POST metodları ile topicler ile iletişim kurmamızı sağlar.

Confluent aynı şekilde github dan "kafka-rest" projemizi kullanmamıza izin verir. (https://github.com/confluentinc/kafka-rest)
