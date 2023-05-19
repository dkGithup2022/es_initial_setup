
## ES 설치와 운영을 위한 설정

리눅스 환경에서의 es 운영을 위해 공통적으로 필요한 요소들에 대해 다룬다.
리눅스가 절반, Elsaticsearch 가 절반 정도 내용에 들어갈 예정 .

</br>

db 운영을 위한 storage, 외부접속을 위한 network, 그리고 환경변수가 제대로 먹었는지 확인하는 그런걸 작성할 것이다. 



</br>

### linux

모든 DB 나 어플리케이션을 사용하기 위한 공통 적인 리눅스 설정이다. Elastic search 는 일종의 DB 이기 때문에 스토리지를 제대로 확보하는 것이 우선이 되어야 한다.

관련 링크 : https://github.com/dkGithup2022/centos_storage_issue

(방치된 db의  linux storage 가 100 % 가 되었을 때, 서버 살리기  )

##### storage 



##### network 

#### env variable 

</br>

### config

##### networkd

##### naming 

##### JVM 


##### ETC 





## ES 운영 현황 조사 

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


