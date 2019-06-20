---
layout: default
title: Heap Dump analizi
date: 22.03.2019
---





> ​	The term Heapdump describes the JVM mechanism that generates a dump of all the live objects that are on the Java heap, which are being used by the running Java application.
> 
> ​	There are two dump formats, the text or classic heap dump format and the Portable Heap Dump (PHD) format. Although the text or classic format is human-readable, the PHD format is compressed and is not human-readable. Both formats contain a list of all object instances in the heap, including each object address, type or class name, size, and references to other objects. These heap dumps also contain information about the version of the JVM that produced the dump. They do not contain any object content or data other than the class names and the values (addresses) of the references.

Java uygulamamız bir kaynak sorunundan ötürü patlayabilir. Bu gibi durumlarda uygumanın heap dump almasını sağlayabilir ve bunu analiz edebiliriz. Öncelikle bu gibi bir durumda heap dump almasını sağlayabilmek için uygulamamızı çalıştırırken;

```java
 -XX:+HeapDumpOnOutOfMemoryError
```

argümanını yazarız, bu şekilde JVM heap'i bittiğinde heap dump oluşturur. Bu dosyayı The Eclipse Memory Analyser (MAT)  ile analiz edebiliriz. Ayrı bir program olarak çalışabildiği gibi aynı zamanda bir eclipse tooling aracıdır. Eclipse eklenebilir. 

MAT'ı download edip istediğimiz klasöre açıyoruz. Açmadan önce dump analiz etmek için bolca RAM'e ihtiyacımız vardır. MemoryAnalyzer.ini dosyasını açıp ram'i arttırmamız lazım. -vmargs degerini 8000 yapalım veya komut satırından çalıştırırken ise aynı işlemi;

```bash
MemoryAnalyzer -vmargs -Xmx8g
```

olarak set edebiliriz. İşyerimde IBM'in J9 JVM'ini kullandığım için heap dump'ı kendi standartına göre alıyor. IBM JVM'i  .phd uzantılı dosyalar oluşturuyor. Bu dosyaları da MAT kullanarak açmak için, MAT'ı açıp help- install new software kısmından IBM toolunu indiriyoruz. (Site address aşağıda)

```
IBM Diagnostic Tool Framework for Java - 
http://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/runtimes/tools/dtfj/
```

Ardından sorunlu phd dosyamı açıyorum. Leak suspect kısmında zaten belleği dolduran sınıfı direk olarak görebildim ve sorunu tespit ettim. Bu tool şu işe yarıyor.


- Öncelikle bu tool ile sınıfın değerlerini okuyamadım, hep sıfırdı. Sebebi heapdump'lar nesne içeriğini içermezler! Nesne değişkenlerinin içeriklerine bakmak için *.dmp uzantılı dosyaları* açman lazım, fakat çok RAM'e ihtiyaç var.
- Heap de hangi sınıftan kaç adet var, ne kadar belleği kaplıyor, sınıfın bellek adresleri vs. vs bilgileri
- Bu sınıflar içinde arama yapmak için SQL benzeri bir OQL özelliği vardır ve heap içinde sorgu yapılabilir!
- Histogram 'a tıklanarak JVM içindeki bütün sınıflar sayısına göre listelenebilir ve sınıfa sağ tıklayarak birçok işlem yapılabilir. 
  - List objects - with outgoing referances dersek, ilgili sınıfın bellekteki bütün instanceları ve adresleri,
  - merge shortest paths to gc roots dersek sınıfı kullanan diğer sınıfları listeleriz 
  - leak suspects ise leak olabilecek şüpheli sınıfları direk kendi yorumlayıp gösteriyor.
  - Eğer JVM ile aynı ortamda çalışıyorsa MAT, çalışan JVM den dump alabilir. Dosya - acquire dump diyerek dump alabiliriz. IBM JVM i de options dan alabiliyoruz.

> ## Generating a heap dump
> 
> Most of my clients run on the Sun(Oracle) JVM, so I will try to keep this post focussed on the instructions for the Sun JVM. In case your application server runs out of memory you can instruct the JVM to generate a heap dump when an OOME occurs. This heap dump will be generated in the HPROF binary format.
> 
> You can do this in by:
> 
> 1. Manually: by using 'jmap', which is available since JDK 1.5.
>    
> 2. Automatically by providing the following JVM command line parameter: `-XX:+HeapDumpOnOutOfMemoryError`
> 
> The size of the heap dump is around the same size as the configured max heap size JVM parameter. So if you have set your max heap size to -Xmx512m , your heap dump will be around that size.
> 
> ## Analyzing the heap dump
> 
> Now you have the heap dump and want to figure out what was inside the heap at the moment the OOME occurred. There are several Java heap dump analyzers out there, where most of them can do more then just heap analysis. The products range from commercial to open source and these are the ones that I tried with my 4Gb .hprof file:
> 
> - [Yourkit](http://www.yourkit.com/)>   
> - [jHat](http://download.oracle.com/javase/6/docs/technotes/tools/share/jhat.html)
> - [Eclipse Memory Analyzer (MAT)](http://www.eclipse.org/mat/)
> -  [Visual VM](https://visualvm.dev.java.net/)
> 
> I was surprised to see that most of the above applications were unable to handle a file of this size. Eclipse Memory Analyzer was actually the only heap dump analyzer that was able to handle a heap dump of this size on my MacBookPro. All the other analyzers were unable to handle a file of this size. Apparently some of these tools tried to load the entire file into memory. On a laptop with only 4 Gb this will not fit.  Eclipse MAT was able to analyze this file within 5-10 minutes.
