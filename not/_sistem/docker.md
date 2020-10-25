---
layout: default
title: Docker
date: 11.02.2018
---


Windows 7 de container'ı çalıştırdıktan sonra portlara bağlanamama sorunu olduğunu öğrendim. Sorunu aşmanın en basit yolu Oracle VirtualBox içinden linux imaja port forwarding tanımlamaktı. Zaten ssh portunun yönlendirildiğini orada göreceksin.
Bu çalışma ile beraber dockerın sanal ağ adresi olarak 172.16 lı bir ip aldığını gördüm.

Hangi portu forward edeceğini netstat -an ile ve ssh portunu inceleyerek tespit edebilirsin.

```sh
docker network ls
docker network inspect bridge
docker network inspect br0
docker network create --subnet 192.168.0.0/16 --gateway 192.168.0.1 --opt com.docker.network.bridge.default_bridge=true --opt com.docker.network.bridge.enable_icc=true --opt com.docker.network.bridge.enable_ip_masquerade=true --opt com.docker.network.bridge.host_binding_ipv4=0.0.0.0 --opt com.docker.network.bridge.name=docker0 --opt com.docker.network.driver.mtu=1500 br0
```

## Docker websphere imajı

IBM in hazırladığı websphere imajını kullanalım;

```sh
docker run --name was -h was --network br0 -p 9043:9043 -p 9443:9443 -p 9080:9080 -d ibmcom/websphere-traditional:8.5.5.11-profile
```

Container'ı ayağa kaldırtıktan sonra was'ın scriptleri ile yönetebiliriz. Önce root olarak login olalım;

```sh
docker exec -u 0 -it was bash
```

```sh
wsadmin.sh -conntype NONE
$AdminTask changeFileRegistryAccountPassword { -userId wsadmin -password wsadmin }
$AdminTask importWasprofile { -archive /tmp/was.car }
$AdminConfig save
```

Administration consola bu adresten ulaşabilirsin;

https://localhost:9043/ibm/console/login.do

Diğer komutlar;

```sh
manageprofiles.sh -backupProfile -profileName AppSrv01 -backupFile /tmp/fileBack

manageprofiles.sh -restoreProfile 

docker cp was:/tmp/a.txt a.txt
docker cp a.txt was:/tmp/a.txt 

sudo -s ile
echo "" > $(docker inspect --format='{{.LogPath}}' was)
```

