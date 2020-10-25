---
layout: default
title: ELK Stack (Elasticsearch, Logstash, and Kibana)
date: 24.10.2020
---


Filebeat çeşitli kaynaklarıda oluşan logları toplayıp Logstash veya Elasticsearch'e gönderen ajanlardır. Şu komutla "filebeat.yml" dosyasındaki config'e göre logları işlemeye başlar.

     ./filebeat -c filebeat.yml -e

"filebeat.yml" dosyasında inputları bu adresteki glob pattern'lara göre tanımlayabiliriz.

- [https://www.elastic.co/guide/en/logstash/current/glob-support.html](https://www.elastic.co/guide/en/logstash/current/glob-support.html)

Websphere için aşağıdaki adımları yaptım;

1. elasticsearch-7.6.0\bin\elasticsearch.bat

2. kibana-7.6.0-windows-x86_64\bin\kibana.bat

   - [http://localhost:5601/status](http://localhost:5601/status)

3. Logstash açılışta hata verirse, logstash-7.6.0\config\jvm.options dosyasında aşağıdaki parametrelerdeki yorumlarını kaldır.

   - -Duser.language=en

   - -Duser.country=US

4. logstash-7.6.0\config\pipelines.yml

   - ```yaml
       - pipeline.id: my_logs
         queue.type: persisted
         path.config: "../config/logstash.conf"
      ```
      
   - Bu kısımda logları bilgisayara persist ediyoruz. Sadece kontrol et.

5. logstash-7.6.0\config\logstash.conf (ident lere dikkat et)

   - ```bash
      input {
        beats {
          port => 5044
        }
      }
      
      output {
        elasticsearch {
          hosts => ["http://localhost:9200"]
          #index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
          #user => "elastic"
          #password => "changeme"
        }
      }
      
      filter {
          if [path] =~ "messagesErr.log" {
              grok {
                match => ["message", "\[%{DATA:wastimestamp} %{WORD:tz}\] %{BASE16NUM:was_threadID} (?<was_shortname>\b[A-Za-z0-9\$]{2,}\b) %{SPACE}%{WORD:was_loglevel}%{SPACE}%{GREEDYDATA:message}"]
                overwrite => [ "message" ]
                tag_on_failure => ["logParseHatasi"]
              }
              grok {
                pattern_definitions => { "WAS_HATA_KODU" => "[A-Z0-9]{9,10}" }
                # match => ["message", "(?:\b%{WAS_HATA_KODU:was_error_code}[:,\s\s\b])?"]
                match => ["message", "(?:.*%{WAS_HATA_KODU:was_error_code}[:,\s\s])?" ]
                overwrite => [ "message" ]
                tag_on_failure => ["WAS_HATA_KODU_Hatasi"]
              }
              grok {
                pattern_definitions => { "LOG4J_HATA_KODU" => "[A-Z]+" }
                match => ["message", "(?:%{LOG4J_HATA_KODU:log4j_error_type}[\s]\[[0-9\s:]+\]\s%{NOTSPACE:log4j_error_line}\s-\s%{JAVACLASS:log4j_error_class})?"]
                overwrite => [ "message" ]
                tag_on_failure => ["LOG4J_HATA_KODU_Hatasi"]
              }
              grok {
                pattern_definitions => { "CAUSE_BY" => ".*Caused by:" }
                match => ["message", "(?:%{CAUSE_BY:caused_by_oncesi} %{NOTSPACE:caused_by})?"]
                overwrite => [ "message" ]
                tag_on_failure => ["CAUSE_BY_Hatasi"]
              }
              grok {
                match => ["message", "(?:%{JAVACLASS:errorClass})?"]
                overwrite => [ "message" ]
                tag_on_failure => ["JAVACLASS_Hatasi"]
              }
          }
      }
      
      ```

6. logstash-7.6.0\bin\logstash -t  komutu ile konfigurasyon kontrol edilir. "Configuration OK" mesajını almamız lazım.

7. logstash-7.6.0\bin\logstash

8. bin\logstash -f erkan-pipeline.conf --config.reload.automatic 

9. bin\logstash -f first-pipeline.conf --config.test_and_exit

10. filebeat-7.6.0-windows-x86_64\filebeat.yml

11. ```yaml
       output.logstash:
         hosts: ["localhost:5044"]
       ```

12.   ```yaml
       filebeat.inputs:
       - type: log
         enabled: true
         # Paths that should be crawled and fetched. Glob based paths.
         paths:
           - \\1.1.1.1\SystemEr?.log
         multiline:
           pattern: '^\[[0-9]+/[0-9]+/[0-9]+[ ][0-9]+:[0-9]+:[0-9]+:[0-9]+[ ][A-Z]+\][ ][0-9A-Za-z]+[ ]SystemErr[ ]+[A-Z][ ][a-zA-Z0-9\[]|[*]+[ ]Start'
           negate: true
           match: after
         tags: ["sysErr"]
         
       - type: log
         enabled: true
         paths:
           - \\1.1.1.1\SystemOut?.log
         multiline:
           pattern: '^\[|[*]+[ ]Start'
           negate: true
           match: after
         tags: ["sysOut"]
         
       processors:
       - dissect:
            tokenizer: "%{}\\%{}\\%{ip}\\%{}\\%{}\\%{}\\%{jvmname}\\%{logfilename}"
            #tokenizer: "\\\\%{sunucuip}\\%{}\\%{jvmname}\\%{logfilename}"
            field: "log.file.path"
            target_prefix: ""
       
       ```

13. Burada multiline her bir log tipi için birden fazla log satırı elde etmek için kullanılır. Buradaki negate ve match alanlarına göre seçim yapılır. Bu alanlar; 

       ```
       negate:false
       
       after
       pattern ile eşleşen satırlar, pattern'a uymayan önceki satırın sonuna eklenir.
       before 
       pattern ile eşleşen satırlar, pattern'a uymayan sonraki satırın önüne eklenir.
       
       
       negate:true
       
       after
       pattern ile eşleşmeyen satırlar, pattern'a uyan önceki satırın sonuna eklenir.
       before 
       pattern ile eşleşmeyen satırlar, pattern'a uyan sonraki satırın önüne eklenir.
       ```


14. Multiline Test etmek için https://play.golang.org/p/uAd5XHxscu 

15. processors alanında ise filebeat'in ürettiği "log.file.path" değişkenini parçalara ayırıp başka bir değişken olarak dışarı expose ediyoruz.

16. filebeat-7.6.0-windows-x86_64\filebeat -c -e filebeat.yml

17. Kontrol etmek için şu adrese git.

    - http://localhost:9200/filebeat-*/_search?pretty

18. Filebeat parse ettiği dosyayı cache'ine alır ve tekra göndermez. Testler sırasında aynı veriyi tekrar tekrar göndermek istersek kaynak dosyayı her seferinde değiştirebiliriz. Veya filebeat-7.6.0-windows-x86_64\data\registry\filebeat klasörünü silerek filebeat in verileri tekrar göndermesini sağlayabiliriz.

19. Bu pipeline'ı oluştururken hataları sistem bazlı test etmek işi çok hızlandırıyor. Ekte filebeat ve logstash için örnek debug dosyaları mevcut. Mesela filebeat çıkışını şu şekilde konsola basabilirsin; 

    ```sh
    output.console:
      pretty: true
    ```

    Ayrıca logstash için ise; 

    ```
    output {
        stdout { codec => rubydebug }
    }
    ```

    diyerek konsola basabiliriz bilgiyi, ayrıca girişi ise şu şekilde elle verebiliriz;

    ```sh
    input {
          generator {
            lines => [
        "[2/13/20 16:45:45:363 TRT] 00000163 SystemErr     R SRVE0014E: Uncaught service() e2d",
        "[2/14/20 17:27:39:729 TRT] 000000a0 PrivExAction  W   J2CA0144W: No mappingConfigAlias found"
    				]
            # Emit all lines 3 times.
            count => 1
            add_field => ["path","C:\Users\te323sw\messagesErr.log"]
          }
        }
    ```
 

## Kibana

Kibana loglarında saatleri farklı göreceksin. Sebebi verilerin kaynağı olan Elasticsearch'ün bütün zaman değerlerini UTC olarak saklamasıdır. Düzeltmek için Kibana da timezone bilgisini gireriz ve görüntülerken ilgili timezone'a göre timestamp zamanları görüntülenir.  

Kibana arayüzüne girip Management - Advanced Settings - timezone değerini browser'ın timezonu'u yerine +3 Europe/Istanbul yapabiliriz. Bu ayar saklanır, restart'da değişmez. Menüye girmişken en altta "Usage Data" kısmını kapatabilirsin, boşa veri göndermesin.

## Kibana log arama

Arama yapmak için "alan_adı:veri" formatı kullanılır. Mesela "message:veri" burada değeri veri olmayanlar için "NOT message:veri" kullanılabilir ve arka arka bu pattern'lar AND ve OR ile eklenebilir. İçinde geçen bir değeri bulmak için ise "message:\*veri\*" kullanılabilir. 


