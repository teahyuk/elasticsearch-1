#목차
## 4. 데이터 검색
### 4.1 검색 api
- 검색 질의 표현 방식
- URI 검색
- Request Body 검색

### 4.2 Query DSL 이해하기
- Query DSL 쿼리의 구조
- Query DSL 쿼리와 필터
- Query DSL의 주요 파라미터

### 4.3 Query DSL의 주요 쿼리
- Match All Query
- Match Query
- Multi Match Query
- Term Query
- Bool Query
- Query String
- Prefix Query
- Exist Query
- Wildcard Query
- Nested Query

### 4.4 부가적인 검색 Api
- 환경설정
- Search Shards API
- Multi Search API
- Count API
- Validate API
- Explain API
- Profile API

## 6. 고급 검색
### 6.1 한글 형태소 분석기 사용
### 6.2 검색 결과 하이라이트하기
### 6.3 스트립트를 이용해 동적으로 필드 추가
### 6.4 검새 템플릿을 이용한 동적 쿼리 제공
### 6.5 별칭을 이용해 항상 최신 인덱스 유지하기
### 6.6 스냅샷을 이용한 백업과 복구

## 7. 하늘 검색 확장 기능
### 7.1 Suggest API 소개
### 7.2 맞춤법 검사기

*****

# 내용 정리
## 4. 데이터 검색
### filter vs query
- query
    - 문서가 쿼리와 얼마나 유사한지를 스코어로 계산
    - 질의가 요청될 때마다 루씬을 이용해 계산을 수행
    - 캐싱되지 않고 디스크 연산을 함
- filter
    - 쿼리의 조건과 문서가 일치하는지를 구분
    - 스코어를 계산하지 않음
    - 자주 사용되는 필터의 결과는 캐시
    - 메모리 연산을 하기 때문에 상대적으로 빠름
### match phrase query
- match 는 순서를 검색 키워드에 대한 순서를 보장하지 않음
- match phrase query는 키워드에 대한 순서를 보장
### 페이징
- from, size를 이용하여 페이징 가능
- 일부의 문서만을 검색하더라도 전체 문서를 읽음
- scroll을 사용할 경우 페이지 표시 없이 데이터를 연속으로 가져올 수 있음
    - https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html
### 정렬
- sort 에 정렬하고 싶은 필드를 이용하여 정렬 가능
- score 가 default
### minimum_should_match
- 몇개 이상의 단어가 일치할 경우에만 검색에 걸리는지를 설정
### fuzziness
- 오차 범위의 글자수 만큼도 검색에 걸리도록 하는 옵션
### boost
- 해당 필드가 매칭되었을 경우 score 을 더 높게 설정
### 검색 환경설정
- adaptive replica selection
    - 검색 샤드 선택을 효율적으로 하기 위해 생긴 옵션
    - 기존 라운드 로빈 방식의 샤드 선택에서 변경됨
    - 빠를것이라고 판단되는 샤드에 검색 요청을 넣는 방식
### 7.0
- 관련 blog
    - https://www.elastic.co/blog/elasticsearch-7-0-0-released
- adaptive replica selection default
- Faster top k retrieval
    - https://www.elastic.co/blog/faster-retrieval-of-top-hits-in-elasticsearch-with-block-max-wand
    - top hits 에 대해 속도 증가를 위한 알고리즘 변경
    - 해당 알고리즘으로 인한 변화
        - score는 더이상 minus 값을 갖지 못함
        - 검색에서의 전체 문서 갯수(total hit) 가 항상 정확한 값을 보장하지 않음
    - track_total_hits 옵션을 이용여 성능 향상을 볼 것인지 아니면 정확한 count를 얻을 것인지를 결정할 수 있음

## 6. 고급 검색
### script
- script를 이용하여 검색시 필드 변경이 가능함
- painless를 밀어주고 있음
- 스크립트를 사용하는 방식은 두가지
    - config 폴더에 스크립트를 저장하는 방식
    - api를 호출할 때 스크립트를 직접 정의해서 사용하는 방식
    - 두번째 방식이 더 많이 사용된다고 하는데 이유는 모르겠음
- 스크립트는 1분에 15줄만 컴파일 되기 때문에 동적으로 사용할 경우 에러 발생할 가능성 있음
    - https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html#prefer-params
### 하이라이트
- 검색 쿼리 할 때 highlight 에 필드를 넣으면 해당 필드의 내용이 테그에 감싸져 나옴
- 필드 내용의 일부만 나옴
- <em> 이 defaul이며 해당 태그는 변경 가능 
### 스넵샷
- 데이터를 백업하는 기능
- 인덱스의 모든 내용을 스넵샷으로 만들어 저장
- yml 설정 변경 필요
- 인덱스의 데이터를 하나의 저장 공간에 저장하기 때문에 외부 repository 를 만들거나 아니면 저장공간이 큰 내부 서버를 이용해야 함