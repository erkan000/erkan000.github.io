---
layout: default
title: WAS built-in cache
date: 12.04.2019
---



Cache ile çalışmak için WAS'ın built-in bir cache manager'ı vardır. "services/cache/distributedmap" JNDI path'inden ulaşabilirsin. İçine istediğin nesneyi koyabilirsin. Kullanımı;

```java
		try{
			InitialContext ctx = new InitialContext();
			DistributedMap cacheObject = (DistributedMap) ctx.lookup("services/cache/distributedmap");
			if(cacheObject.containsKey(dk)){
				int adet = (int)cacheObject.get(dk);
				adet++;
				cacheObject.put(dk, adet);
			}
			
		}catch(Exception e){
			log.error(e);
		}
```

Bu servis cluster ortamında veriyi paylaştırmıyor. Sadece aynı JVM içindeki uygulama bu nesneye ulaşabiliyor. Çözümü "hazelcast", "redis" vs..

Recursive almak için aşagıdaki metodu netten buldum fakat sonsuz döngüye giriyor. Araştırmadım.

```java
	public static void recursiveGet(String name, PrintWriter out){
		try{
			InitialContext c = new InitialContext();
			System.out.println("NameSpace = -------" + c.getNameInNamespace());
	        	NamingEnumeration<NameClassPair> list = c.list(name);
			while (list.hasMoreElements()) {
				NameClassPair nameClassPair = list.nextElement();
				String name2 = nameClassPair.getName();
				String type = nameClassPair.getClassName();
				log.info((name.equals("") ? name : "") + "/" + name2 +"\t\t" + type);
			}
		} catch(Exception e){
			log.error(e);
		}
	}
```




