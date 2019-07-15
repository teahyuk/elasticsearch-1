## elasticsearch

### elasticsearch?
 - 루씬 기반의 실시간 검색을 제공하는 오픈소스 분산검색엔진

### elasticsearch 특징
 - 일반적으로 많은 양의 데이터를 색인해서 전문검색과 실시간 통계 목적으로 사용한다.
 - 검색의 유사성을 이용하여 오타처리, 검색제안 등의 기능을 제공한다.
 - 데이터를 색인하고 검색하는 루씬의 기능들에 쉽게 접근하도록 해준다.
 - REST API를 통해 기능을 제공하고 JSON을 사용한다.
 - RDB와 다르게(row단위) 데이터를 문서 단위로 저장한다.
 - 자동으로 데이터를 샤드에 나눠서 클러스터의 이용 가능한 서버들 간에 균형을 유지한다.

### elasticsearch 활용
 - 검색기능 외에도 집계기능을 제공하여 실시간 분석엔진으로도 활용 가능하고 데이터를 저장할 수도 있어 NoSQL 저장소로도 활용할 수 있다.
 - 관계형 데이터베이스와도 사용가능.
 - MongoDB 같은 문서 기반 저장소와도 사용가능.
 - 기본 백엔드로 사용 
    - 데이터 저장기능 + 검색엔진기능
 - 기존 시스템에 추가하여 사용 
    - 기존 DB + 검색엔진기능 
    - 데이터 저장소로써 요구하는 모든 기능을 항상 제공하지 않을수도 있음.
    - elasticsearch를 추가하여 기존 시스템과 함께 동작하도록 
 - 기존도구와 함께 사용
    - logstash, flume, kibana 같은 도구들과 함께 사용

### Apache 루씬? 
 - Java 기반의 고성능 검색엔진 라이브러리

### elasticsearch(2010~) vs 솔라(2004~)
 - 솔라 또한 루씬기반의 오픈소스 분산 검색엔진이다.
 - 루씬과 솔라는 2010년에 하나의 아파치 프로젝트로 합쳐짐.
 - 두 검색 엔진 모두 유사한 기능을 제공하고 빠르게 개발되고있음.
 - elasticsearch는 scale out 분산모델을, 솔라는 샤딩, distributed 분산의 측면이다. (?)
 - 각각 서로가 가지고 있지 않은 기능들을 가지고 있어서 선택은 결정하는 시점에 필요한 특정 기능에 좌우될수 있다.
 - 핵심기능들은 비슷하니 취향차이라고함. 지금도 그럴지는 미지수.

### elasticsearch 용어정리
 1. aggregation : 집계
 2. | elasticsearch | RDBMS |
    |-------|-------|
    | index | database | 
    | type | table |
    | document | row  |
    | field | column |
    | mapping | schema  |
 3. mapping : 각각 field를 정하여 data의 Type을 정하는것.
 4. node : elasticsearch를 실행하는 프로세스, elasticsearch의 인스턴스
 5. shard : index 의 일부분, 하나의 루씬 index, 역 색인을 포함하는 파일들의 모음
 6. cluster : 클러스터
 7. replica : 복제
 8. index : 색인
 9. replica : 주 샤드의 복사본
 10. chunks : elasticsearch index는 shard라는 chunks로 나뉜다.