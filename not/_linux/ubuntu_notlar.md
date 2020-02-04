---
layout: default
title: Linux notlarim
date: 24.01.2020
---

## Windows ile paylaşım yapma
EXT4 ile formatlanmış bir disk üzerinde klasöre sağ tıkla “Local Network Share” tabı içinde bu klasörü paylaş de ve anonymous erişime izin ver sadece. Bu kadar windowsdan \\192.168.2.100 diyerek ulaşabilirsin.

## İşletim sistemi güncelleme
```sh
sudo apt-get update 
sudo apt-get upgrade
```
- (red-hat tabanlı ise yum, deb uzantılı dosyalar debian ve ubuntu sistemlerde dpkg programı ile kurulabilirler.)
- (rpm uzantılıları ubuntuya yüklemek için bunu deb uzantılıya çeviren alien programı kullanılabilir.)
- Kullanılmayan yükleme dosyalarını silmek;
```sh
sudo apt autoremove
```

## Program çalıştırma;
Önce ll komutu ile dosya çalıştırılabilir mi ona bak. Ubuntu'da yeşil renk oluyor. Değil ise ```chmod +x dosya_adi``` diyerek dosyayı çalıştırılabilir yaparız. Sonra da `./dosya_adi` diyerek çalıştırırız.

## Komut satırından arama yapmak;
```sh
sudo find / -iname 'aranacak_dosya' -print
```

## Apache web server kurmak;
- sudo apt-get install apache2 apache2-utils -y
komutu ile (red-hat tabanlılarda ise httpd) root klasörü bilgisayarda (/var/www/html) klasörü içindedir. Konfigurasyon dosyası ise; (/etc/apache2/apache2.conf) dir.
- grep -i documentroot /etc/apache2/apache2.conf
komutu ile kolayca edit edilebilir.

## Tomcat kurmak;
(https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-ubuntu-16-04 )
```sh
sudo apt-get remove apache2
sudo apt-get update
sudo groupadd tomcat
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
cd /tmp
curl -O http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.15/bin/apache-tomcat-8.5.15.tar.gz
sudo mkdir /opt/tomcat
sudo tar xzvf apache-tomcat-8*tar.gz -C /opt/tomcat --strip-components=1
cd /opt/tomcat
sudo chgrp -R tomcat /opt/tomcat
sudo chmod -R g+r conf
sudo chmod g+x conf
sudo chown -R tomcat webapps/ work/ temp/ logs/
sudo update-java-alternatives -l
sudo nano /etc/systemd/system/tomcat.service
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl status tomcat
sudo systemctl enable tomcat
sudo nano /opt/tomcat/conf/server.xml
sudo systemctl restart tomcat
```

tomcat dizini sahibini değişttirmek için;
```sh
sudo chown -R chi /opt/tomcat/
```

## Tomcat 8080 portunu 80. porta yönlendirmek;
```sh
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo /etc/init.d/netfilter-persistent save 
```

## Bir servisin durumunu görüntülemek, servis başlatmak, servis durdurmak;
```sh
service apache2 status
sudo service apache2 stop
sudo service apache2 start
sudo service apache2 restart
```

## ISO imajı USB ye yazarak boot edilebilir USB oluşturmak.
```sh
sudo fdisk -l
```
komutu ile USB diskin harfi tespit edilir. Önce komut çıktısına bakıp sonra USB yi takarsan daha emin olursun harfinden.
```sh
sudo dd if=kali-linux-2016.1-amd64.iso of=/dev/sdb bs=512k
```
komutu ile de dosya USB'ye kopyalanır. 512 kopyalama hızıdır. Tavsiye edilen.

## Java Kurmak.
Tar.gz uzantılı dosya oracle sitesinden indirilir.
```sh
tar -xvf jdk-8u91-linux-x64.tar.gz 
sudo mkdir -p /usr/lib/jvm 
sudo mv ./jdk1.8.0_91 /usr/lib/jvm/ 
sudo update-alternatives --install "/usr/bin/java" "java" 	"/usr/lib/jvm/jdk1.8.0_91/bin/java" 1 
sudo update-alternatives --install "/usr/bin/javac" "javac" 	"/usr/lib/jvm/jdk1.8.0_91/bin/javac" 1 
sudo update-alternatives --install "/usr/bin/javaws" "javaws" 	"/usr/lib/jvm/jdk1.8.0_91/bin/javaws" 1 
sudo chmod a+x /usr/bin/java 
sudo chmod a+x /usr/bin/javac 
sudo chmod a+x /usr/bin/javaws 
sudo chown -R root:root /usr/lib/jvm/jdk1.8.0_91/ 
sudo update-alternatives --config java 
```
son konuttan sonra başka alternatif java yok ise hata verebilir. Var ise şu komutlar ile de diğerleri için de yapılır.
```sh
sudo update-alternatives --config javac
sudo update-alternatives --config javaws
```

java -version komutu ile kontrol edilebilir.

## Sistemi temizlemek;
- sudo apt-get autoremove
- sudo apt-get autoclean

## Maven Yüklemek;
- sudo apt-get install maven
- mvn -version

## MySql Server Yüklemek;
- sudo apt-get update
- sudo apt-get install mysql-server
- sudo mysql_secure_installation

- Çalışıp çalışmadığına bakmak için 		systemctl status mysql.service
- Çalıştırmak için ise				sudo systemctl mysql start

## Mysql tablo yedeklemek;
Veritabanındaki bir şemayı ve içindeki bütün tabloları yedeklemek için;
mysqldump -u root -p0503 fav 

komutu kullanılabilir. Burada şifre -p parametresi ile şifre arasında boşluk bulunmamalıdır.”fav” ise şemanın adıdır. 

Bu komut ekrana basar, günün tarihi ile zipli bir şekilde kaydetmek için;
mysqldump -u root -p05031 fav | gzip > /home/chi/database_`date '+\%d-\%m-\%Y'`.sql.gz

komutu kullanılır. Buradaki `date '+%d-%m-%Y'` komutu günün tarihi ile dosyayı kaydetmek içindir. Cron ile bunu hergün yapabiliriz, cron yazısına bakınız. (% işareti hataya sebeiyet veriyor. Bu sebepten backslash ile escape ettik.)

## Cron , zamanlanmış görev oluşturmak;
/etc/crontab	dosyasını aç. İçine şu satırı ekle;
15 2 	* * * 	root 	mysqldump -u root vs vs…
Burada ilk 5 parametre minute, hour, day of month, month, day of week dir. * ise any anlamına geliyor. Yukarıdaki örnek her gün gece 2:15 o komutu çalıştır demek. 
Sonraki root ise hangi kullanıcı ile bu komutun çalıştırılacağını belirler.
Sonraki ise komutun kendisidir.
/etc/init.d/cron restart komutu ile cron tekrar başlatılır ve değişiklikler çalışır.

Crontab -e	komutu ile kullanıcıya ait görevleri görebiliriz.

Escaping backslash (\) ile yapılır.

Mesela her gece mysql veritabanında fav şeması yedeğini almak için aşağıdakini kullandım.
mysqldump -u root -pXXX fav | gzip > /home/chi/etc/dbBackUp/database_`date '+\%d-\%m-\%Y'`.sql.gz

## Diskteki boş alanı görmek için;
df -h

diskleri görmek için; 
lsblk

## Bir servisi yeniden başlatmak;
sudo systemctl restart tomcat

## Kullanıcıyı bir gruba eklemek;
sudo usermod -a -G tomcat chi	
sudo usermod -a -G groupName usrName

## Ubuntu teması değiştirme;
sudo apt-get install unity-tweak-tool
unity-tweak-tool

programını kullanarak “Theme” başlığı altında temayı değiştirebilirsin. Numix olan güzel olmuş.
GNOME arayüzü kullanıyorsan ise gnome-twek-tool kullanabilirsin.

Değişiklikten sonra terminali renklendirmek istersen ise;

 1685  gedit ~/.bashrc 
içine en son satıra şunu ekle.
PS1='\[\033[1;36m\]\u\[\033[1;31m\]@\[\033[1;32m\]\h:\[\033[1;35m\]\w\[\033[1;31m\]\$\[\033[0m\] '

 1686  source ~/.bashrc

## Icon teması değiştirmek için ise Numix in önce icon setini yüklüyoruz.
sudo add-apt-repository ppa:numix/ppa
sudo apt-get update
sudo apt-get install numix-gtk-theme numix-icon-theme-circle

ve yukardaki tweak tool ile “Icon” yazan kısımdan Numix çıkacak ve onu seçebiliriz artık.

## Dosya sahipliği değiştirmek
Bir klasörü ve içindeki bütün dosyaların sahipliğini başka bir kullanıcıya devretmek için;
	sudo chown -R tomcat:tomcat .
Komutu kullanılır. -R parametresi recursive için, ilk tomcat kullanıcı ikinci tomcat ise grup parametresidir.

## Dosya yetkileri değiştirmek

	sudo chmod -R 777 14subat

## Sistemin “Enviroment Variables” görüntülemek;
printenv 			bütün değişkenleri görüntüler
printenv PATH		sadece PATH değişkenini görüntüler
echo $PATH	PATH 		değişkenini farklı bir yoldan görüntüler
(https://help.ubuntu.com/community/EnvironmentVariables )

## Auto start-up
Kullanıcı login olduğunda otomatik bir program çalıştırmak için; Dash’i açıp “Startup Applications” yaz. Buradan istediğin programı ekleyebilirsin. Bu ekran arka planda ~/config/autostart klasörü içine programAdi.desktop isimli bir dosya yaratmaktadır.

## Kullanıcının yüklediği programlar;
İki yolu vardır. Birincisi ubuntu software center da “installed” kısmında bir kısmını bulabiliriz. İkinci yöntem ise şu komutlardır;
	grep " install " /var/log/dpkg.log
	grep " install " /var/log/dpkg.log.1
	zgrep " install " /var/log/dpkg.log.2.gz
var/log içinde kaç adet zip varsa o kadar bu komutu koştururuz.

## Wget ile bir klasör çekmek;
	wget -m -np -c --no-check-certificate -R "index.html*" "https://theswissbay.ch/pdf/Gentoomen%20Library/Programming"

## Wireless networkleri listeleme;
	sudo iwlist wlan0 scan

## cURL komutu;
curl komutu desteklediği bazı protokoller ile data transferi yapmaya yarar. Bunu yaparken libcurl kütüphanesini kullanır. Wget e benzer fakat ikisinin de olumlu ve olumsuz tarafları vardır. HTTP POST isteği yapılabilir, TLS desteği vardır, HTTP cookie leri gönderilebilir ve HTTP auth yapılabilir.
	curl	http://x.y/a.zip			Gelen veriyi ekrana yazar.
	curl -O http://x.y/a.zip		Büyük o serverdaki isimle download 
	curl -o b.zip http://x.y/a.zip	Küçük o b.zip isimle download eder.
	curl -h					ile diğer parametreleri görebiliriz.

## Docker Yüklemek;
Önce Docker reposunu repolara ekliyoruz ve ap-get install ile kuruyoruz.
1.) Repoya eklemek;
	sudo apt-get update
	sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

2.) Docker kurmak;
	sudo apt-get update
	sudo apt-get install docker-ce
3.) Docker test etmek için;
	sudo docker run hello-world
4.) uninstall etmek için;
	sudo apt-get purge docker-ce
komutu, fakat bu komut imageları silmediği için, 
	sudo rm -rf /var/lib/docker
komutu ile images, containers, and volumes silinir.

Her seferinde sudo kelimesini kullanmamak için; kullanıcıyı kurulumla gelen docker grubuna eklemeliyiz.

Kullanıcıyı yeni bir gruba eklemek;
Bir kullanıcının grupları id komutuyla görüntülenir;
	id -nG
Kullanıcını yeni bir gruba eklemek için ise;
	sudo usermod -aG docker chi
docker grup adı, chi user adıdır. Parametre ise kullanıcının hali hazırdaki gruplarına append et demektir.


## Docker Compose Yüklemek;
Önce yukarıdaki gibi docker’ı yüklüyoruz. Docker kurulumu bittikten sonra;
1.) Önce docker compose son sürümü şu adresten kontrol edilir.
	https://github.com/docker/compose/releases

	
2.) Docker kurmak; Aşağıdaki komutta sürüm numarası son sürüm ile değiştirilir ve komut çalıştırılarak localimize compose çalıştırılabilir dosyası download edilir. 
	sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

indirilen dosya çalıştırılabilir yapılır.
	sudo chmod +x /usr/local/bin/docker-compose
	
3.) Docker compose test etmek için;
	docker-compose –version

4.) Unistall etmek için sadece dosyayı sil, o kadar;
	sudo rm /usr/local/bin/docker-compose



## Sistem bilgileri, versiyon, sürüm;
	uname -a		Kernel bilgileri
veya
	sudo cat /etc/os-release
veya
	lsb_release -a
veya
	cat /proc/version

## Docker Komutları;
- docker info				container sayısı, image sayısı, docker klasörü
- docker container ls		bütün containerları listele(imajın bir instance'ı)
- docker image ls			bütün imajları listeler
- docker images				bütün imajları listeler
- docker ps					sadece çalışan containerları
- docker ps -a				bütün containerları listele
- docker run image			imaj localde varsa çalıştırır yoksa download edip çalıştırır.
- docker run hello-world		hello-world ü container olarak çalıştırır.
- docker run –it Ubuntu bash	-it = interactive mode sanal ubuntuyu çalıştırır 						ve içindeki komut satırına düşer
- docker run –it centos /bin/bash	interaktif diğer bir örneği
- docker run -p 8080:8080 image	imaj içindeki 8080 inci portu gerçek makinamızın 						8080'ine bağlar.
- docker pull image			imajı docker hub'dan locale indirir.
- docker rmi ImageID		imajeID li imajı siler
- docker inspect image		imaj bilgilerini gösterir.

Burada kaldım, diğer komutlar devam ediyor.
https://www.tutorialspoint.com/docker/docker_working_with_containers.htm

- docker logs db2			db2 isimli container loglarına bak
- docker ps -a			bütün containerları listele, çalışan,duran
- docker container start xx	duran containerı başlat
- docker container restart xx	çalışanı tekrar başlar.
- docker container rm xx		container’ı sil
- docker volume ls			volume’leri listele
- docker volume prune		kullanılmayan volume’leri siler
- docker system prune 		duran container, volume vs. siler

## Docker ile WAS çalıştırmak;
Önce docker imajını docker hub dan bulup çekeriz, birçok versiyon var, benimki 11 olduğu için 11 versiyonunu çekeceğim. Bu aşamada was imajını internetten indirecektir.
	docker pull ibmcom/websphere-traditional:8.5.5.11-profile
çalıştırmak için ise;
	docker run --name was -h was -p 9043:9043 -p 9443:9443 -d ibmcom/websphere-traditional:8.5.5.11-profile
Buradaki –name parametresi container’a kısa bir isim vermek için, -h ise hostname’i set etmek için, -d ise konsolu arka planda deattach mode da kullanılıyor.  Artık sunucumuz hazırdır. Burada profile tag’ı önceden ibm tarafından hazırlanmış default bir profil olduğunu belirtmek için kullanılıyor. Ardından bu profile ait “wsadmin” kullanıcısı şifresini bulmamız gerekiyor. 
	docker exec was cat /tmp/PASSWORD
komutu ile wsadmin şifresini öğreniyoruz ve test etmek için https://localhost:9043/ibm/console/login.do?action=secure adresine gidip login oluyoruz. 
Sunucu loglarını görmek için;
	docker logs -f --tail=all was
Docker container içine girmek için;
	docker exec -it was bash
diyerek sunucumuz içinde gezebilir, onun lokalindeki farklı sunucuda ayar yapabiliriz. Burada was programlarına 
	cd /opt/IBM/WebSphere/AppServer/profiles/AppSrv01/bin
klasöründen ulaşabiliriz. 
Sunucu içinde iken wsadmin şifresini değiştirmek için, şu komut ile admin consola girilir,
	wsadmin.sh -conntype NONE
Ardından şu iki komut ile şifre değiştirilir.
	$AdminTask changeFileRegistryAccountPassword {-userId wsadmin -password wsadmin}
	$AdminConfig save

Sunucuyu başlatma ve durdurma için ise çeşitli yollar mevcuttur.

	docker stop --time=60 was
	docker start was
	/opt/IBM/WebSphere/AppServer/profiles/AppSrv01/bin/startNode.sh

Normalde ./startServer.sh server1 ve ./stopServer.sh server1 ile yapılıyor?

## Docker ile db2

	docker run -itd –	name db2 -e DBNAME=testdb -v ~/:/database -e DB2INST1_PASSWORD=password -e LICENSE=accept -p 50000:50000 --privileged=true ibmcom/db2


	database 	:	testdb
	username	:	db2inst1
	password	:	password

Bilgisayarı yeniden başlatınca durur DB. O yüzden önce duran container’ı bul, sonra başlat, dosya sistemine yazdığı için kaldığı yerden 2 dakikada açılıyor.

	docker ps -a
	docker start 066a3430a997
	docker logs db2

## Docker compose kullanımı
	compose ile birden fazla image ile çalışabilir ve bunları tek seferde yönetip, aralarında network kurabliriz. Compose çalışmak için “docker-compose.yml” veya “docker-compose.yaml” dosyasını arar. Aşağıda örneği vardır;
	
my-test:
 image: hello-world

İlk satır container’ımızın ismidir, ikinci satır ise container içindeki imajlarımız. Bu dosyayı kaydettiğimiz klasörde şu komutu vererek container’ı başlatabiliriz.
	docker-compose up

Çalıştırdıktan sonra çalışan servisleri görüntülemek için yaml dosyamızın bulunduğu dizinde;
	docker-compose ps

Yaml içindeki bütün servisleri silmek için;
	docker-compose down
	docker-compose down –volumes		(Volume lerle birlikte sil)

Örnek bir docker-compose dosyası;

version: '3.3'

services:
  db:
    image: mysql:5.7
    restart: always
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: wordpress

  wordpress:
    image: wordpress
    restart: always
    volumes:
      - ./wp_data:/var/www/html
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: password
    depends_on:
       - db

volumes:
    db_data:
    wp_data:


Açıklamaları ise;
In the first line, we are specifying the Compose file version. There are several different versions of the Compose file format with support for specific Docker releases.
Next, we are defining two services, db and wordpress. Each service runs one image and it will create a separate container when docker-compose is run.
The db service:
    • Uses the mysql:5.7 image. If the image is not present on the system it will be pulled it from the Docker Hub public repository.
    • Uses the restart always policy which will instruct the container to always restart.
    • Creates a named volume db_data to make the database persistent.
    • Defines the environment variables for the mysql:5.7 image.
The wordpress service:
    • Uses the wordpress image. In the image is not present on your system Compose will pull it from the Docker Hub public repository.
    • Uses the restart always policy which will instruct the container to always restart.
    • Mounts the wp_data directory on the host to /var/lib/mysql inside the container.
    • Forwards the exposed port 80 on the container to port 8080 on the host machine.
    • Defines the environment variables for the wordpress image.
    • The depends_on instruction defines the dependency between the two services. In this example, db will be started before wordpress.


## Klasör boyutu
	sudo du -hs /var/lib/docker