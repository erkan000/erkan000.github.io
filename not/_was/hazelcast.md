---
layout: default
title: Hazelcast
date: 16.04.2019
---

https://www.baeldung.com/java-hazelcast

Hazelcast, im memory data grid ürünü olarak geçiyor. Verilerin bellekte saklayýp farklý JVM'lerde paylaþmayý, hatta ayarlanýrsa farklý sunucular arasýnda bile paylaþmayý saðlýyor. Hazelcast veri daðýtýmý için geliþtirilmiþ bir kütüphanedir. Processler arasý iletiþimde kullanabileceðiniz gibi verinizi tek bir lokasyon üzerinde cache’lemek için de kullanabilirsiniz. Bir java client oluþturup instance'larýn birine baðlanýp içindeki verileri çekebiliriz. Management center ile görsel ayar yapabiliriz. Kullanmak için "hazelcast-3.11.jar" ve "hazelcast-client-3.11.jar" jarlarýný projemizse ekliyoruz. Oluþturmak için;

	HazelcastInstance hz = Hazelcast.newHazelcastInstance();

satýrý yetiyor. Burada bir config vererek birsürü özellik açabiliriz. Ýçine herhangi bir nesne konabiliyor. Veri okuyup yazmak için;
	
	Map<Integer, Integer> map = hz.getMap("veri_ismi");
	Integer veri = hz.get("int_veri");

gibi kullanýmlarý mevcuttur. Aþaðýda örnekler var. Bir de web uygulamasý içinde kullanýyor isek, web uygulamasý destroy olurken hazelcast'i de kapatmak gerekiyor. Bunun için context destroy metodunu implemente ederken hazelcast'i kapatmalýyýz. Ýlgili kod þu þekildedir;

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

