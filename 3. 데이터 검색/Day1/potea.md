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


