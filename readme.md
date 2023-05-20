## ES 설치와 운영을 위한 설정

리눅스 환경에서의 es 운영을 위해 공통적으로 필요한 요소들에 대해 다룬다.
리눅스가 절반, Elsaticsearch 가 절반 정도 내용에 들어갈 예정 .

</br>

db 운영을 위한 storage, 외부접속을 위한 network, 그리고 환경변수가 제대로 먹었는지 확인하는 그런걸 작성할 것이다. 



</br>

### linux

모든 DB 나 어플리케이션을 사용하기 위한 공통 적인 리눅스 설정이다. Elastic search 는 일종의 DB 이기 때문에 스토리지를 제대로 확보하는 것이 우선이 되어야 한다.

관련 링크 : https://github.com/dkGithup2022/centos_storage_issue
- (방치된 db의  linux storage 가 100 % 가 되었을 때, 서버 살리기  )



##### storage 

</br>

1. 스토리지 있는지 확인 하기

fdisk -l : 물리적으로 장착된 디스크 스토리지 크기를 확인.  (1TB 의 충분한 디스크 스토리지가 보인다.)


```
[root@localhost /]# fdisk -l

Disk /dev/sda: 966.4 GB, 966367641600 bytes, 1887436800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c2eea

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200  1887436799   942668800   8e  Linux LVM

Disk /dev/mapper/centos-root: 53.7 GB, 53687091200 bytes, 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 6308 MB, 6308233216 bytes, 12320768 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-home: 905.3 GB, 905290186752 bytes, 1768144896 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

```


</br>


2.  df : 파일 시스템에서 사용 할 수 있는 디스크 크기를 확인

      (centos -home 에서 880 GB 사용이 가능함을 확인, 이정도면 충분하다)

```
[root@localhost /]# df
Filesystem              1K-blocks    Used Available Use% Mounted on
devtmpfs                  6057144       0   6057144   0% /dev
tmpfs                     6068992       0   6068992   0% /dev/shm
tmpfs                     6068992   25452   6043540   1% /run
tmpfs                     6068992       0   6068992   0% /sys/fs/cgroup
/dev/mapper/centos-root  52403200 3112532  49290668   6% /
/dev/sda1                 1038336  198408    839928  20% /boot
/dev/mapper/centos-home 883640772   33216 883607556   1% /home
tmpfs                     1213800       0   1213800   0% /run/user/0

```

</br>

3. es 위치 변경

ES 의 위치를 df 에서 확인한  바꾼 뒤  확인 
```agsl
[root@localhost elasticsearch]# pwd
/home/elasticsearch
[root@localhost elasticsearch]# ls
LICENSE.txt  NOTICE.txt  README.asciidoc  bin  config  data  elasticsearch-7.14.1  jdk  lib  logs  modules  plugins
```

systemctl service script 도 경로를 바꿔준다. 
```agsl
[Unit]
Description=Elasticsearch Cluster
Documentation=https://www.elastic.co/kr/products/elasticsearch
Wants=network-online.target
After=network-online.target

[Service]
RuntimeDirectory=elasticsearch-7.14.1
WorkingDirectory=/home/elasticsearch

LimitMEMLOCK=infinity
LimitNOFILE=65535
LimitNPROC=4096

ExecStart=/home/elasticsearch/bin/elasticsearch
ExecReload=/home/elasticsearch/bin/elasticsearch reload
RestartSec=3

User=elastic
Group=elastic

```

</br>

##### network 

9200 포트 ( 혹은 어플리케이션에 사용할 포트 )에 대한 방화벽을 열고, 외부 연결을 위한 elasticsearch 의 통신 관련 config 에 대해 말한다.
( dhcp 나 기본적인 리눅스 컴퓨터의 인터넷 연결에 대해선 다루지 않는다. )

9200 포트 열기
```
firewall-cmd --permanent --zone=public --add-port=9200/tcp
```

잘 됐는지 한번 보기

```agsl
firewall-cmd --list-all-zone

```


elasticsearch/config/elasticsearch.yml 

호스트 0.0.0.0 으로 해야 외부 접속 가능하다.

나머지는 single-node 기준으로 다음에 할때 복붙하려고 올림 

```agsl
network.host: 0.0.0.0
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#
http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
discovery.type: single-node
discovery.seed_hosts: ["127.0.0.1"]
```




#### ETC

##### 자바 버전 및 ENV

자바는 11버전 이상 ( es 7.x 버전 기준  )

</br>

##### ES 폴더와 파일의 소유자와 그룹 옵션 변경

chown -R elastic:elastic 으로 하위까지 다 elastic 소유자로 변경

```
root@localhost home]# ls -al
합계 0
drwxr-xr-x.  4 root    root     42  5월 19 16:20 .
dr-xr-xr-x. 18 root    root    241  5월  2 12:42 ..
drwx------.  3 elastic elastic  76  5월  2 14:00 elastic
drwxr-xr-x. 11 elastic elastic 195  5월  2 14:00 elasticsearch
```

</br>


##### max open files 

/etc/security/limits.conf
```
#<domain>      <type>  <item>         <value>
#
#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

root hard nofile 500000
root soft nofile 500000
elastic hard nofile 500000
elastic soft nofile 500000
# End of file
```

확인하기  :cat /proc/sys/fs/file-max
```agsl
[root@localhost security]# cat /proc/sys/fs/file-max
1192299
```


#####  max_thread 값 변경


</br> 


</br>

##### memory swap option 

스 왑 방 지 
-> 힙사이즈 고정되어 있을때 최적화가 잘됨. disk io 는 검색에서 성능의 주된 병목 요인임


/{elastic}/config/elasticsearch.yml
```agsl

# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.

```



</br>


### config


##### JVM

전체 램사이즈의 절반정도의 HEAP SIZE 를 고정 할당 .

나머지 절반은 커널 작업을 위해 남겨야함 

/{elastic}/config/jvm.options

```
################################################################
## IMPORTANT: JVM heap size
################################################################
##
## The heap size is automatically configured by Elasticsearch
## based on the available memory in your system and the roles
## each node is configured to fulfill. If specifying heap is
## required, it should be done through a file in jvm.options.d,
## and the min and max should be set to the same value. For
## example, to set the heap to 4 GB, create a new file in the
## jvm.options.d directory containing these lines:
##
-Xms5g
-Xmx5g

```

##### ETC 

기억나는 건 이정도 ? 

나머지는 기억날때 업데이트 하겠음 . 





## ES 운영 현황 조사 - Opster article 번역 

출처 : https://opster.com/blogs/elasticsearch-best-practices-3000-cluster-analysis/

opster 에서 올라온 글이다. es cloud 환경의 운영사례를 조사하여, 사람들이 자주 하고 있는 현황과 실수들에 대해 작성되어 있다. 위의 글에 대한 번역, 중간에 추가하고 싶은 내용도 조금은 추가 했다. 
( by dankim )

</br>
</br>


##### 리소스 관련 현황과 권장 사항

1. 사용중인 하드웨어 리소스
   1. 평균 메모리 사용량 : 40%
   2. 평균 disk 사용량: 43 %
   3. 70% 이상의 디스크 사용 노드 : 18%


</br>
   

2. 운영 모범 사례
   1. 클러스터 이름을 기본값('elasticsearch')으로 사용하지 않는다.
      
        - 멀티 클러운영환경에서 식별이 제대로 되지 않음
        - 나중에 수평적 확장 ( 노드 추가 ) 시에 구분이 되지 않음
          
        </br>
      
   2. 와일드 카드 사용을 제한 해라

      </br>
   3. 쿼리 시 스크립트 사용을 부분적으로 제한해라 
      
      - 성능적 이슈의 주요 원인이 될 수 있다. 
      - 특히 스크립트 쿼리에서 정규식 활성화를 disable 하는 것이 권장된다. 만약 정규식을 활성화한다면 해당 클러스터의 리소스 모니터링과 성능 테스트를 빡세게 하는 것을 추천함.

</br>
</br>

##### 노드 관련 현황과 권장 사항

1. 노드 설정 현황

   1. 30 % 의 유저가 올바르지 않은 minimum_master_node config 로 운영을 하고 있다.
   
   2. 16 % 의 유저가 dedicated cordinate 를 충분히 갖고 있지 않다.

   3. 12 % 의 유저가 dedicated master node 를 충분히 갖고 있지 않다. 

   4. 15 % 의 유저가 서킷 브레이커에 대한 exception 을 결험하고 있다.  

   5. 사용자의 5.1 % 가 높은 대기열로 인해 성능의 하락을 경험한다.

</br>


##### 노드 관련 현황과 권장 사항

1. 디스크와 워터마크 

   워터 마크는 es 에서 제공하는 disk 관리에 대한 기능이다.

   </br>

   일정 % 의 disk 사용량을 넘긴다면 샤드를 추가 생성 않는다던가, 인덱스의 write 작업을 막는다.

   </br>


   기본값이 enabled : true 로 되어 있어 워터마크 사용을 끈 유저는 적지만, water mark 임계값을 부분적으로 초과한 클러스터는 5% 정도 발견되었다.

</br> 

한국어 설명 링크 : https://medium.com/@zoo5252/elastic-search-%EB%94%94%EC%8A%A4%ED%81%AC-%EC%9A%A9%EB%9F%89-%EC%A0%9C%ED%95%9C-55e58e51ce85


