---
layout: default
title: Hazelcast
date: 16.04.2019
---

https://www.baeldung.com/java-hazelcast

Hazelcast, im memory data grid �r�n� olarak ge�iyor. Verilerin bellekte saklay�p farkl� JVM'lerde payla�may�, hatta ayarlan�rsa farkl� sunucular aras�nda bile payla�may� sa�l�yor. Hazelcast veri da��t�m� i�in geli�tirilmi� bir k�t�phanedir. Processler aras� ileti�imde kullanabilece�iniz gibi verinizi tek bir lokasyon �zerinde cache�lemek i�in de kullanabilirsiniz. Bir java client olu�turup instance'lar�n birine ba�lan�p i�indeki verileri �ekebiliriz. Management center ile g�rsel ayar yapabiliriz. Kullanmak i�in "hazelcast-3.11.jar" ve "hazelcast-client-3.11.jar" jarlar�n� projemizse ekliyoruz. Olu�turmak i�in;

	HazelcastInstance hz = Hazelcast.newHazelcastInstance();

sat�r� yetiyor. Burada bir config vererek birs�r� �zellik a�abiliriz. ��ine herhangi bir nesne konabiliyor. Veri okuyup yazmak i�in;
	
	Map<Integer, Integer> map = hz.getMap("veri_ismi");
	Integer veri = hz.get("int_veri");

gibi kullan�mlar� mevcuttur. A�a��da �rnekler var. Bir de web uygulamas� i�inde kullan�yor isek, web uygulamas� destroy olurken hazelcast'i de kapatmak gerekiyor. Bunun i�in context destroy metodunu implemente ederken hazelcast'i kapatmal�y�z. �lgili kod �u �ekildedir;

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

