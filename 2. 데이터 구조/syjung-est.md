## 데이터 구조
![데이터_구조](../img/데이터구조_0.PNG)

### 논리적 배치(Application 관점) : 문서(document), 타입(type), 색인(index)
![논리적 배치](../img/데이터구조_1.jpg)
 - document
    - index, 검색의 최소한의 단위
    - JSON 형식의 document
        ~~~
        {
            "name" : "Elasticsearch Denver",
            "organizer" : "Lee",
            "location" : [
                "name" : "Denver, Colorado, USA",
                "geolocation" : "39.1231, -104.1241"
            ]
        }
        ~~~
    - document엔 스키마가 존재하지 않는다. 어떠한 형식이든 document 를 가질수 있다. -> schema-free

 - type
    - 테이블이 행에 대한 컨테이너인것과 같이 document 에 대한 논리적인 컨테이너다. -> document 모임

 - index
    - mapping type의 컨테이너이다. -> 다소 비슷한 특성을 가진 document의 모음
    - DB와 같이 독립적인 document덩어리 이다.
    - 디스크에 같은 파일 집합으로 저장됨.
    - refresh_interval 설정으로 간격 정의 가능. : refresh 비용은 꽤 비싼편으로 default 1초 (NRT)
    - 한개 혹은 N개의 샤드로 구성할 수 있다.
    - 하나 이상의 주 샤드와 0개 이상의 레플리카 샤드로 구성된다.

### 물리적 배치(관리자 관점) : 노드(node), 샤드(shard)
 - 어떻게 elasticsearch가 확장하는지 이해할 수 있다.
![물리적 배치](../img/데이터구조_2.jpg)
 - node
    - elasticsearch를 실행하는 프로세스
    - elasticsearch의 인스턴스
    - index들의 모임

 - shard
    - elasticsearch가 다루는 가장 작은 단위, 하나의 샤드는 하나의 루씬 색인 루씬 색인은 역 색인을 포함하는 파일들의 모음
    - index를 분할한 조각들.
    - 색인을 생성할 때 원하는 샤드 수를 간단히 정의할 수 있음
    - 각 샤드는 그 자체가 온전한 기능을 가진 독립적인 "index"이며, 클러스터의 어떤 노드에서도 호스팅할 수 있다.
    - 너무 적은 샤드는 확장에 제한을 주고 너무 많은 샤드는 성능에 영향을 준다.
    
### split brain
 - 클러스터의 두 파트가 통신할 수 없어서 다른 파트가 떨어져 나갔다고 판단하는 현상.
 - 노드가 충분히 빨라 서로간의 통신을 할 수 있도록 보장해야함.

### replica
 - 주 샤드의 정확한 복사본이다.
 - 항상 생성하거나 제거할 수 있다.
 - 검색성능과 장애복구에 이유를 둔다.

### 매핑(mapping)
 - 매핑정보 보기
~~~
 index/_mapping/~~
~~~
 - elasticsearch는 모든 필드와 타입, 그리고 다른 설정에 대한 매핑을 보관하고있다. 이 매핑은 index의 type마다 다르다.
 - 타입에서 지금까지 색인한 모든 문서의 모든 필드를 포함한다. 하지만 모든 문서가 모든 필드를 가질 필요는 없다.
 - 새로운 document가 매핑에 존재하지 않는 필드와 함께 색인하면 elasticsearch 는 자동으로 새로운 필드를 매핑에 추가한다.
 - 필드를 추가하기위해 타입이 무엇인지 추측하여 매핑한다.
  ex> "age" : 14 -> long 이군?
 - 필드를 자동으로 찾고 그것에 맞게 매핑을 조정한다.
 - 타입으로부터 document에 있는 모든 필드를 포함하고, document의 필드를 어떻게 색인할 것인지 elasticsearch에게 알려준다.
