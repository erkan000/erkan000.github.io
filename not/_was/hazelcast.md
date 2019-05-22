---
layout: default
title: Hazelcast
date: 16.04.2019
---


https://www.baeldung.com/java-hazelcast

Hazelcast, im memory data grid ürünü olarak geçiyor. Verilerin bellekte saklayıp farklı JVM'lerde paylaşmayı, hatta ayarlanırsa farklı sunucular arasında bile paylaşmayı sağlıyor. Hazelcast veri dağıtımı için geliştirilmiş bir kütüphanedir. Processler arası iletişimde kullanabileceğiniz gibi verinizi tek bir lokasyon üzerinde cachelemek için de kullanabilirsiniz. Bir java client oluşturup instance'ların birine bağlanıp içindeki verileri çekebiliriz. Management center ile görsel ayar yapabiliriz. Kullanmak için "hazelcast-3.11.jar" ve "hazelcast-client-3.11.jar" jarlarını projemizse ekliyoruz. Oluşturmak için;

	HazelcastInstance hz = Hazelcast.newHazelcastInstance();

satırı yetiyor. Burada bir config vererek birsürü özellik açabiliriz. ıçine herhangi bir nesne konabiliyor. Veri okuyup yazmak için;
	
	Map<Integer, Integer> map = hz.getMap("veri_ismi");
	Integer veri = hz.get("int_veri");

gibi kullanımları mevcuttur. Aşağıda örnekler var. Bir de web uygulaması içinde kullanıyor isek, web uygulaması destroy olurken hazelcast'i de kapatmak gerekiyor. Bunun için context destroy metodunu implemente ederken hazelcast'i kapatmalıyız. ılgili kod şu şekildedir;

@WebListener
public class MedulaContextListener implements ServletContextListener{


	@Override
	public void contextDestroyed(ServletContextEvent sce) {
		try {
			hz..getLifecycleService().shutdown();
			//MedulaGenelParametreler.instance.getHazelcast().getLifecycleService().shutdown();
		} catch (MedulaException e) {
			e.printStackTrace();
		}
	}

