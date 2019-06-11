---

layout: default
title: Cache sistemleri
date: 10.06.2019

---



Gün geçtikte uygulamalarımız iş kuralları genişler, yaptıkları iş çoğalır ve uygulamamızı kullanan kullanıcı sayıları artar. Bu artış sistemlerimiz üzerinde kaynakların tükenmesine ve uygulamamızın hizmet veremeyecek duruma gelmesine yol açabilir. Genellikle uygulamalardaki dar boğaz veritabanıdır ve veritabanının da bir kapasitesi vardır. Değerli olan veritabanı kaynaklarını daha az kullanmak için günümüzde "Caching sistemleri" kullanılır. Bu sistemler sık kullandığımız ve zamanla az değişen verileri ön bellekte tutar, uygulama veritabanına gitmek yerine bu sistemlerden okur ve böylelikle veritabanı daha az kullanılarak uygulama performansı arttırılmış olur. Cache sistemlerini açıklamak için veritabanı üzerinden örneği verdim fakat cache sistemleri sunucu session bilgilerini tutarak veya sık ziyaret edilen static web sayfalarını bile cacheleyerek sunucu performansını arttırmak için kullanılabilmektedir.

Dünyada kullanılan en önemli cache sistemleri şunlardır;

1. [Memcached](https://memcached.org/)	( Distributed memory object caching system, RAM'i tamamen doldurmaz, eski verilerin üzerine yazar)
2. [Ehcache](https://www.ehcache.org/) ( Gerektiğinde verileri persist edebilir, hibernate destekliyor.)
3. [Redis](https://redis.io/) ( Gerektiğinde verileri persist edebilir, tek bir master vardır, diğerleri verileri ondan çeker.)
4. [Hazelcast](https://hazelcast.org/) ( Master yoktur, P2P çalışır.)
5. [Websphere Extreme Scale](https://www.ibm.com/support/knowledgecenter/en/SSTVLU_8.6.1/com.ibm.websphere.extremescale.doc/cxsoverview.html) ( IBM'in caching çözümü ) 

Her sistemin farklı özellikleri vardır[^1], dolayısıyla seçim yazılımcının ihtiyacına göre belirlenebilir. Bu yazıda hazelcast'i tanıtacağız çünkü MEDULA uygulaması 4 fiziksel sunucu ve 80 JVM üzerinde clustered çalışmaktadır. Cache'e yerleştirdiğimiz her verinin diğer JVM'ler tarafından da ulaşılabilmesini istiyoruz. 

## Hazelcast

Hazelcast java için In-Memory Data Grid platformudur. Clustered ortamlarda veri paylaşımını ve ölçeklenebilirliği sağlamaktadır. Sunucu ve port ayarı yapılarak cache'e attığımız verinin JVM'ler arasında paylaşılması ve senkronizasyonunu otomatik olarak yapabilmektedir. Open source ve enterprise versiyonları bulunmaktadır. Projemize Hazelcast'i eklemek için pom.xml dosyamıza aşağıdaki dependency'yi ekliyoruz;

```xml
		<dependency>
			<groupId>com.hazelcast</groupId>
			<artifactId>hazelcast</artifactId>
			<version>3.12</version>
		</dependency>
```



Hazelcast'i ayağa kaldırmak için uygulamamızda ister uygulama çalışınca, ister ilk istekte Hazelcast nesnemizi oluştururuz. Yeni bir hazelcast nesnesi şu şekilde oluşturulur;

```java
HazelcastInstance hzInstance = Hazelcast.newHazelcastInstance();
```

Nesne oluşturulunca konsolda şuna benzer nesne id sini görebilirsiniz;

```sh
Members {size:1, ver:1} [
	Member [10.206.18.25]:5701 - d8abccec-a50d-48e2-8764-5d405e162b0c this
]
```

Yukarıda da görüldüğü üzere hazelcast instance'ımız 10.206.18.25 ip adresli makinanın 5701 nolu portunda çalışmaya başladı. Diğer bir JVM de aynı kod bloğu çalıştığı anda iki JVM birbirlerini ile konuşarak otomatik olarak senkron olacak ve aşağıdaki gibi konsolda görülebilecektir;

```
Members {size:1, ver:1} [
	Member [10.206.18.25]:5701 - d8abccec-a50d-48e2-8764-5d405e162b0c
	Member [10.206.18.30]:5702 - d1fserfs-w4sf-2sd3-6321-3ea3fsds2w3s this
]
```

Nesneyi bir web uygulamasında oluşturuyorsak, uygulamamızın ServletContextListener sınıfının destroy metodunda Hazelcast'i de kapatmamız gerekmektedir.

```
hzInstance.getLifecycleService().shutdown();
```



Uygulamamız aynı fiziksel sunucuda ise her oluşturulan Hazelcast nesnesi diğer JVM'de çalışan nesne ile kendi kendine haberleşebilir. Fakat uygulamamız birden fazla fiziksel sunucuda ise hazelcast'e diğer sunucular ile konuşması gerektiğini söyleyebiliriz. Yukarda hazelcast nesnesini oluştururken parametre olarak Config sınıfını parametre olarak gönderirirz ve config sınıfında network ayarlarını yapabiliriz. Aşağıdaki Config sınıfında 2 fiziksel sunucu ip adresi belirtilmiş ve hazelcast'in kullanacağı port başlangıç numarası 6001 olarak belirtilmiştir. 

```java
public static Config getConfig() {
	Config config = new Config();
	config.setInstanceName("medula");
	NetworkConfig network = config.getNetworkConfig();
	network.setPort(6001);
	network.setPortAutoIncrement(true);
	JoinConfig join = network.getJoin();
	TcpIpConfig tcpIpConfig = join.getTcpIpConfig();
	tcpIpConfig.addMember("10.206.18.25").setEnabled(true);
	tcpIpConfig.addMember("10.206.18.30").setEnabled(true);
	join.getMulticastConfig().setEnabled(false);
    return config;
}
```
Gördüğünüz üzere 2 node'dan oluşan bir cluster tanımı yapılmış ve hazelcast'in kullanacağı port olarak 6001 set edilmiştir. Bu konfigurasyon ile Hazelcast nesnesi oluşturmak için ise;

```java
HazelcastInstance hzInstance = Hazelcast.newHazelcastInstance(getConfig());
```

şeklinde oluşturabiliriz. Artık hazelcast instance'ımız çalışmaya başladı ve içine veri atıp okuma işlemleri yapabiliriz. Aşağıda adı "data" olan distributed map oluşturulmaktadır. getMap metodu adı data olan map'i arar eğer yok ise oluştururur, var ise halihazırdaki nesneyi döndürür. 

```java
Map<String, Integer> map = hzInstance.getMap("data");
```

Artık elimizde map değişkeni bulunmaktadır. java.util.Map arayüzünü implemente eden bu nesne ile map'e okuma ve yazma için kullandığımız metodları kullanabiliriz.





[Hazelcast IMDG Reference Manual](https://docs.hazelcast.org/docs/3.12/manual/html-single/index.html#hazelcast-imdg-reference-manual)

[Hazelcast Samples](https://github.com/hazelcast/hazelcast-code-samples)



[^1]: http://bilgisayarkavramlari.sadievrenseker.com/2012/11/07/caching-mekanizmalari/