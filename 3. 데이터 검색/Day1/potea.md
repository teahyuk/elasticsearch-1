# 엘라스틱서치 실무가이드

## 03 데이터 모델링

### 엘라스틱서치 분석기
- 루씬을 기반으로 구축된 텍스트 기반 검색엔진
- 루씬이 제공하는 분석기를 그대로 활용한다.

#### 분석기의 구조
1. 문장을 특정한 규칙에 의해 수정한다.
2. 수정한 문장을 개별 토큰으로 분리한다.
3. 개별 토큰을 특정한 규칙에 의해 변경한다.

- Character Filter
    - 문장을 분석하기 전에 입력 텍스트에 대해 단어를 변경하거나 태그를 제거하는 역할
- Tokenizer Filter
    - 분석기를 구성할 때 하나만 사용 가능하며 텍스트를 어떻게 나눌 것인지 정의
- Token Filter
    - 토큰화된 단어를 하나씩 필터링해서 사용자가 원하는 토큰으로 변환한다.
    - 불필요한 단어제거, 동의어 사전, 영문자 대소문자로 변경


## 04 데이터 검색

- 검색 API
- Query DSL 이해하기
- Query DSL의 주요 쿼리
- 부가적인 검색 API

### 검색 API
- 문장은 색인 시점에 텀(term)으로 분해된다. 검색시에는 이 텀을 일치시켜야 검색이 가능하다.
- 색인 시점 분석기(analyzer)와 분석 시점 분석기를 이용해서 분석 수행

#### 검색 질의 표현 방식
- URI 검색 = HTTP GET 요청 방식
    - 검색조건이 복잡해지면 가독성이 떨어지는 단점
- Request Body 검색 = RESTful API를 이용한 Qurey DSL

### Query DSL 이해하기
- Request Body 검색과 URI 검색 모두 _search API를 이용해서 검색 질의

#### Query DSL 쿼리와 필터

|| 쿼리 컨텍스트 | 필터 컨텍스트 |
|-|-|-|
| 용도 | 전문 검색 시 사용 | 조건 검색 시 사용 |
| 특징 | 연관성 관련(score) 계산 | 연관성 계산을 하지 않음 |

##### 쿼리 컨텍스트
- 문서가 쿼리와 얼마나 유사한지 스코어로 계산
- 내부의 루씬을 이용해 계산 수행 (결과를 캐싱하지 않는다.)
- 전문 검색에 많이 사용
- 디스크 연산을 수행하므로 상대적으로 느리다.

##### 필터 컨텍스트 
- 쿼리 조건과 문서가 일치하는지(Yes/No)를 구분
- 스코어 계산 없이 단순 매칭 여부 확인
- 자주 사용되는 필터의 결과는 엘라스틱서치가 내부적으로 캐싱
- 메모리 연산을 수행하므로 상대적으로 빠르다.

##### Query DSL의 주요 파라미터
- Multi Index 검색
- 쿼리 결과 페이징
- 쿼리 결과 정렬
- _source 필드 필터링
- 범위 검색
- operator 설정
- minimum_should_match 설정
- fuzziness 설정
- boost 설정

### Query DSL의 주요 쿼리

#### Match All Query
- 색인의 모든 문서 쿼리

#### Match Query
- 형태소 분석을 통해 텀(term)으로 분리한 후 텀들을 이용해 검색 질의 수행

#### Multi Match Query
- Match Query와 기본 사용법은 동일하나 여러 개의 필드를 대상으로 검색

#### Term Query
- 별도의 분석작업 없이 입력된 텍스트가 존재하는 문서 검색
- Match Query = Full Text Query
- Term Query = keyword 데이터 타입을 사용하는 필드 검색

#### Bool Query
- 하나 이상의 쿼리를 조합해서 더 높은 스코어를 가진 쿼리 조건으로 검색 수행
- AND, OR, NAND, FILTER

| Elasticsearch | SQL |
|-|-|
| must : [필드] | AND 칼럼 = 조건 |
| must_not : [필드] | AND 칼럼 != 조건 |
| should : [필드] | OR 칼럼 = 조건 |
| filter : [필드] | 칼럼 IN (조건) |

#### Query String
- 내장된 쿼리 분석기를 통한 질의 작성

#### Prefix Query
- 해당 접두어가 있는 모든 문서를 검색

#### Exists Query
- 실제 값이 존재하는 문서만 찾을 경우 사용

#### Wildcard Query
- 와일드 카드(*, ?)와 일치하는 구문을 찾는다.

#### Nested Query
- 분산 데이터 환경에서도 SQL 조인과 유사한 기능을 수행
- 엘라스틱서치는 성능상의 이유로 Parent 문서와 child 문서를 모두 동일한 샤드에 저장

### 부가적인 검색 API
#### 효율적인 검색을 위한 환경설정
- 동일 데이터를 가지고 있는 샤드 중 하나만 선택해서 검색 수행
    - 샤드 선택 방법: 라운드로빈, 동적 분배 방식
- 검색 요청시 타입아웃 설정

#### Search Shards API
- 검색 수행되는 노드 및 샤드 정보 확인

#### Multi Search API
- 여러 건의 검색을 한 번에 요청

#### Count API
- 검색된 문서의 수 카운트

#### Validate API
- 쿼리의 유효성 검사

#### Explain API
- _score 값이 계산된 정보 확인

#### Profile API
- 쿼리에 대한 수행 계획과 수행된 시간을 돌려주는 API
- 성능 튜닝 및 디버깅 시에 유용


# ElasticSearch Cookbook 3rd 
- [Sample.KR](http://www.acornpub.co.kr/book/elasticsearch-cookbook-3)
- [Sample.US](https://github.com/PacktPublishing/Elasticsearch-5x-Cookbook-Third-Edition)

## 05 검색

- 검색 실행
- 정렬
- 하일라이팅
- 스크롤 쿼리 실행
- search_after 기능 사용
- 결과의 inner hits 반환
- 올바른 쿼리 제안
- 매치된 결과 카운트
- explain 쿼리
- 쿼리 프로파일링

### 검색 실행
- HTTP 메소드는 GET or POST
```
    http://<server>/_search
    http://<server>/<index_name(s)>/_search
    http://<server>/<index_name(s)>/<type_name(s)>/_search
```
- 다중 색인과 다중 타입은 쉼표로 구분한다.
- 색인이나 타입을 지정하면 검색은 해당 내용에 한정한다.

### 정렬
- 정렬하는 가장 일반적인 기준은 텍스트 쿼리와의 관련성
- 쿼리에 sort 섹션 추가

### 하일라이팅
- 쿼리로 매치된 텍스트의 요약/강조 할 수 있도록 pre_tags/post_tags 제공
- 다양한 태그로 표시가능하도록 tags_schema 제공

### 스크롤 쿼리 실행
- 쿼리는 실행할 때마다 결과를 만들고 사용자에게 반환
- 전체 도큐먼트를 고유하게 반복하는 특수한 커서를 제공
- 색인 재생성 작업, 매우 큰 결과 집합을 반복하는 경우 유용

### search_after 기능 사용
- from과 size를 사용하는 엘라스틱서치 표준 페이징은 모든 쿼리에 대해 from 값 앞의 모든 결과를 연산하고 버려야 하므로 큰 데이터 집합에서 성능이 매우 나쁨
- search_after는 스크롤 결과를 빠르게 건너뛸 수 있는 기능
    - 루씬 색인에서 모든 텀은 정렬되 저장되므로 루씬이 텀 값을 건너 뛰는 것은 매우 빠르다.
    - search_after는 루씬 검색을 빠르게 건너뛰고, 결과 페이징 속도를 높이도록 쿼리를 작성한다.

### 결과의 inner hits 반환
- 기본적으로 엘라스틱서치는 검색 유형과 매치되는 도큐먼트만 반환하며 중첩 도큐먼트는 반환하지 않는다.
- inner_hits를 사용하면 중첩 쿼리의 중간 결과를 유지하고 유저에게 반환한다.

### 올바른 쿼리 제안
- [Term Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-term.html)
- [Phrase Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-phrase.html)
- [Completion Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-completion.html)
- [Context Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/suggester-context.html)


## 06 텍스트 및 수치형 쿼리

- term 쿼리 사용
- terms 쿼리 사용
- perfix 쿼리 사용
- wildcard 쿼리 사용
- regexp 쿼리 사용
- span 쿼리 사용
- match 쿼리 사용
- query_string 쿼리 사용
- simple_queray_string 쿼리 사용

### term 쿼리 사용
### terms 쿼리 사용
### perfix 쿼리 사용
### wildcard 쿼리 사용
### regexp 쿼리 사용
### span 쿼리 사용
### match 쿼리 사용
### query_string 쿼리 사용
### simple_queray_string 쿼리 사용


## 07 관계 및 지오 쿼리

- has_child 쿼리 사용
- has_parent 쿼리 사용
- nested 쿼리 사용
- geo_bounding_box 쿼리 사용
- geo_polygon 쿼리 사용
- geo_distance 쿼리 사용
- geo_distance_range 쿼리 사용
- geo_hash 쿼리 사용

### has_child 쿼리 사용
### has_parent 쿼리 사용
### nested 쿼리 사용
### geo_bounding_box 쿼리 사용
### geo_polygon 쿼리 사용
### geo_distance 쿼리 사용
### geo_distance_range 쿼리 사용
### geo_hash 쿼리 사용
