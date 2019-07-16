# ElasticSearch Cookbook 3rd 
- [Sample.KR](http://www.acornpub.co.kr/book/elasticsearch-cookbook-3)
- [Sample.US](https://github.com/PacktPublishing/Elasticsearch-5x-Cookbook-Third-Edition)

## 시작하기
- 노드와 클러스터의 이해
- 노드 서비스의 이해
- 데이터 관리
- 클러스터, 복제, 샤딩의 이해
- 일래스틱서치와 통신
- HTTP 프로토콜 사용
- 네이티브 프로토콜 사용

### 소개
- keywords: node, index, shard, type/mapping, document, field

### 노드와 클러스터의 이해
- 마스터 노드
- 보조 노드
- 데이터 노드
- 수집 노드
- 클라이언트 노드: 로컬메모리 캐시

#### 참조
- [REST](https://en.wikipedia.org/wiki/Representational_state_transfer)
- [MapReduce](https://en.wikipedia.org/wiki/MapReduce)

### 노드 서비스의 이해
- 서비스
    - 클러스터, 색인, 매핑, 네트워크, 플러그인, 집계, 인제스트, 언어 스크립팅

### 데이터 관리
- 대량의 레코드를 관리하기 위해 색인을 여러 개의 부분(샤드)으로 분할, 여러 노드로 분산
    - 레코드는 레코드 ID 기반의 샤딩 알고리즘 
    - 대규모 데이터에서 일래스틱서치 성능의 샤드 개수에 따라 수평적으로 확장
- 용어 비교표

| ElasticSearch | SQL | MongoDB |
|---|---|---|
| Index | Database | Database |
| Shard | Shard | Shard |
| Mapping/Type | Table | Collection |
| Field | Columm | Field |
| Object(JSON) | Record (tuples) | Record (BSON) |

#### 참조
- [JSON](https://en.wikipedia.org/wiki/JSON)
- [Shard](https://en.wikipedia.org/wiki/Shard_(database_architecture))

### 클러스터, 복제, 샤딩의 이해
- 최소한 노드 두 개와 레플리카 셋 한 개를 항상 유지

### 일래스틱서치와 통신
- HTTP 프로토콜 사용
- 네이티브 프로토콜 사용

#### 참조
- [Java API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/index.html)

## 다운로드와 설정
- 일래스틱서치 다운로드 및 설치
- 네트워킹 설정
- 노드 설정
- 리눅스 시스템을 위한 설정
    - 파일 디스크립터 제한 변경
    - 메모리 스왑 방지 설정
- 다양한 노드 타입 설정
    - node.master: 마스터 노드
    - node.data: 데이터 노드
    - node.ingest: 수집 노드
- 클라이언트 노드 설정
- 수집 노드 설정
- 일래스틱서치에 플러그인 설치
    - 업데이트시 모든 노드의 버전을 업데이트 해야함
- 플러그인 수동 설치
