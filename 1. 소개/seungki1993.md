### ElasticSearch 특징

- 아파치 루씬 기반
    - 아파치 루신이라는 자바 검색 라이브러리 사용
    - 루씬에서 지원하는 거의 모든 기능을 제공

- 실시간 분석
    - 저장된 데이터를 검색 할 때 별도의 재시작이나 상태 갱신 불필요
    - 색인 작업이 완료된 데이터는 바로 검색 가능

- 분산 시스템
    - 클러스터는 하나 혹은 여러 노드로 바인딩 된다.
    - 노드는 하나의 서버 그 자체가 될 수 도 있고 한 Host에서 여러 노드를 실행 할 수 있다.
    - 시스템 규모가 커지면 새 노드를 실행하여 바인딩 하면 쉽게 시스템을 확장 할 수 있다.
    - 데이터는 하나의 노드가 아닌 여러 노드에 분산 저장 되며(shard) 복사본(replica)을 유지한다
        - 각 데이터가 여러 노드에 분산 되어있고 복사본을 유지하기에 데이터 유실을 방지 할 수 있다.
        
- 멀티 태넌시
    -   서로 다른 인덱스에 있는 데이터를 검색시 하나의 질의로 묶어서 검색 및 결과를 도출 할 수 있다.

- 전문 검색
    - 데이터 전체 문장에서 검색어를 추출하여 저장하고 이를 바탕으로 검색
 
- 데이터 형식 JSON

- RestApi 사용

### 데이터 색인
-   엘라스틱 서치는 루씬 기반으로 색인 절차를 걸쳐 역파일 색인이라는 구조로 저장
    
    - 역파일 색인 (Inverted Index table 자료 구조 사용)
    
        - 데이터는 엘라스틱 서치에 저장 되기 전에 Analyzer을 거친다. Analyzer을 거친 데이터는 각각의 토큰으로 불리 되어
지는데 불리되어진 토큰이 어는 Document에 사용되는지 인덱싱 하는 것이다.
        -  Analyzer는 크게 Character filter , Tokenizer , Token filter로 구성
 
 -  ex) input data Hello world
 
    |검색어 | 문서|
    |-------|-----|
    | hello | doc2,doc1|
    |world  | doc2|
    
### 솔라
- 루씬 기반으로 만들어진 검색 어플리케이션
- 루씬 기반이기에 ElasticSearch와 비교 대상


### MongdDB
- 해당 책에서는 MongoDB와 elasticSearch 비교가 있다.
- 사실 두개의 공통점은 단지 Document Store이지 역할이 다르다
- MongoDB는 데이터 저장 , ElasticSearch 검색

### ElasticSearch vs RDB
|RDB|ElasticSearch|
|---------|-------|
| 데이터 베이스 | 인덱스|
|테이블|타입|
|행|Document|
|열|필드|
|스키마 | 매핑|

- 원래 인덱스마다 여러 타입이 존재 했으나 6버전 부터 인덱스당 type을 한개로 제한
    - 이유 : 한 인덱스에 A type , B type이 있으며 두 개의 type에는 name이라는 필드가 있다고 가정
            두 타입의 name field는 내부적으로 같은 루씬 field를 사용
            즉 두 필드가 타입이 다름에도 서로 관련이 있어 내부적으로 문제 발생
        https://www.elastic.co/guide/en/elasticsearch/reference/master/removal-of-types.html

### 게이트웨이
-   클러스터 재시작시 클러스터 상태 밋 샤드 데이터를 저장하는 저장소
- 마스터 노드에 설정

  
 