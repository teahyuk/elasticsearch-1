## 데이터 검색_Day2

### 사용자지정 분석기
 - 색인이 생성될 때, 특정 색인을 위해 settings 으로 설정
 ~~~
 -XPOST {url}/{index} -d '{
   "settings": {
     사용자 지정 내용
     "analyzer": {
       사용자 지정 분석기
     }
   }
  }'
 ~~~
 - 사용자 지정 내용
    - shard 갯수
    - replica 갯수
 - 사용자 지정 분석기
    - type
    - tokenizer
    - filter
    - char_filter
      - type
      - mappings : ["u=>you", "ph=>f"]
 - elasticsearch의 설정 파일에서 전역 분석기로 설정
    - elasticsearch.yml 에 지정하며 설정 내용은 위의 내용과 비슷
    - elasticsearch 재시작 하여야 설정이 먹힘.

### 사용자지정 분석기 사용    
 - mapping시 analyzer 로 사용자 지정 분석기를 지정한다.
    - not_analyzed로 특정 필드는 분석하지 않도록 지정할 수 있음.
  ~~~
  {
    "mappings": {
      ...
      "properties": {
        "name": {
          "type": "string",
          "analyzer": "myCustomAnalyzer"
        }
      }
    }
  }
  ~~~
 - analyzer옵션 파라미터로 넘길수도 있음.
 - url로 analyzer를 사용할수 있음.

### 분석기
#### 내장 분석기
 - 표준 분석기
 - 단순 분석기
 - 화이트스페이스
 - 불용어
 - 키워드
 - 패턴
 - 언어 및 다국어
 - 스노우볼

#### 토큰화
 - 표준 토크나이저
 - 키워드
 - 문자
 - 소문자화
 - 화이트스페이스
 - 패턴
 - UAX URL EMAIL
 - 경로 계층

#### 토큰 필터
 - 표준 토큰필터
 - 소문자화
 - 길이
 - 불용어
 - TRUNCATE, TRIM, LIMIT TOKEN COUNT
 - Reverse
 - unique
 - ASCII Folding
 - 동의어

#### Ngram, edge ngram, shingle

### 스태밍
 - 단어를 단어의 원형이나 어근으로 줄이는 역할을 한다.
  ex> "administrations" -> "administr", "administrator" -> "administr"
 - 스태밍 알고리즘
    - 스노우볼 필터
    - porter_stem필터
    - kstem필터
  [스태밍 알고리즘_234p 표 5.1]

## 유사도 검색
 - 내가 찾고자 하는 문서와 관련 있는 문서가 처음으로 나오길 바란다.
 - 질의에 대한 문서를 찾는 과정을 "Scoring"이라 한다.

### 문서 점수 계산 방법_TF-IDF
 - 단어가 얼마나 반복되는지와 얼마나 자주 사용되는 단어인지가 점수에 영향을 미친다. 즉, 하나의 문서에서 단어가 여러 번 반복되면 관련성은 높아진다. 하지만 전체 문서에서 단어가 자주 반복된다면, 관련성은 낮아진다. -> TF-IDF(term frequency(단어빈도)/inverse document frequency(역 문서 빈도))

#### TD 단어빈도
 - 단어가 얼마나 자주 문서에서 반복되는가
~~~
"We will discuss Elasticsearch at the next Big Data group."
"Tuesday the Elasticsearch team will gather to amswer questions about Elasticsearch."
~~~
~~~
[Elasticsearch]
첫번째 문서 : TD=1
두번째 문서 : TD=2
~~~
 - 두번째 문서의 점수가 더 높다.

#### IDF 역 문서 빈도
 - 전체 문성서 자주 반복되는 토큰은 덜 중요하다
~~~
"We use Elasticsearch to power the search for our website."
"The developers like Elasticsearch so far."
"The scoring of documents is calculated by the scoring formula."   
~~~
~~~
[Elasticsearch]
 DF=2 : 2개의 문서에서 사용됨
 IDF=1/DF : 1/2
~~~
~~~
[the]
 DF=3 : 3개의 문서에서 사용됨
 IDF=1/DF : 1/3
~~~
 - 역문서 빈도는 단어의 빈도를 균형 잡는 중요 요인이다.
 - 예를들어 사용자가 "the score" 단어를 검색한다고 할때 "the"는 일반 영어 문서에 대부분 포함 되게 되어있기 때문에 "the"의 빈도가 "score"의 빈도를 압도한다.

#### 루씬의 점수 계산 공식
 - TD-IDF에 기초하여 계산된다.
 [루씬의 점수 계산 공식_243p 그림 6.3]
 - 주어진 질의 q와 문서 d의 점수는 문서 d의 텀의 단어 빈도에 대한 제곱근, 텀에 대한 역 문서 빈도의 제곱, 문서에서 필드에 대한 정규화 지수, 텀에 대한 부스트의 곱 (!!!)

### 문서 점수 계산 방법_Okapi BM25
 - 확률 기반 유사도 알고리즘
 - BM25의 3가지 설정
   - k1
      - 단어 빈도가 점수에 얼마나 중요한지 제어한다. (TF)
   - b
      - 0-1 사이의 숫자 형으로 문서의 길이가 점수에 미치는 영향도를 설정한다.
   - discount_overlaps
      - 여러 토큰이 같은 장소에서 발생할 경우 필드에 영향을 미칠지와 어떻게 길이를 정규화할지 elasticsearch에게 알려줄 수 있다. default true

### 부스팅
 - 문서의 관련성을 수정하는 절차
#### 색인 시 부스팅 -> deprecated
#### 질의 시점 부스팅
 - 점수를 계산할 때 부스트 값은 정규화된다.
  ex> 모든 필드에 10이라는 boost 점수를 설정했아면 모든 값이 같으므로 1로 정규화됨. 부스트값은 상대값이다.
 - query_string, match, term 등 쿼리에 자유롭게 사용할 수 있다.
 ~~~
  "boost": {소수}
  "field": ["name^3", "description"]
  "query": "elasticsearch^3 AND \"big data\""
 ~~~

### explain
 - URL이나 전송 내용에 지정하여 사용한다.
~~~
"explain=true"
_explain
~~~
~~~
"_explanation" 필드에 설명이 포함되어 나옴.
~~~
[explain_253p 예제 6.8]
 - 검색이 되지 않았아면 검색되지 않았는지 원인이 나온다.

### function_score
 - 임의의 함수에 숫자로 점수를 지정하여 초기 질의에 맞는 문서의 점수를 결정하는데 세부적으로 조절할 수 있도록 한다.
#### 구조
 - filter 안의 쿼리에 해당하는 내용의 경우 가중치 함수를 이용해서 점수를 증가시킨다.
~~~
"function_score": {
  "query": {
    "match": {
      "description": "elasticsearch"
    }
   },
   "functions": [
   {
    "weight": 2
    "filter": {
      "term": {
        "description": "hadoop"
      }
    }
   },
   {
    "weight": 3
    "filter": {
      "term": {
        "description": "logstash"
      }
    }
   }
   ... N개 =
  ]
 }
~~~

#### score_mode
 - 개별 함수를 어떤 방법으로 결합할지 정한다.
 - multiply | sum | avg | first | max | min
 - 만약 first로 지정한다면 첫번째로 일치하는 function의 점수만 사용한다.
 - 오직 처음에 문서와 일치하는 문서만 2의 가중치를 적용한다.

#### boost_mode (?)
 - 원본 질의 점수와 functions의 점수를 어떻게 결합할지 정한다.
 - sum | avg | max | min | replace

#### functions
 - field_value_factor : 어떤 항목의 점수를 높이기 위해 다른 데이터를 활용하여 점수를 부여한다.
 - script : Groovy를 사용하여 세세하게 조정할수 있게끔 지원
 - random : 무작위점수를 문서에 할당한다.
 - decay 함수 : 특정 필드를 기준으로 점수를 점진적으로 줄여준다.
 [곡선그래프_266, 267p]
    - linear
    - gauss
    - exp
 - 여러 함수들을 함께 사용할 수 있다.

### 필드 데이터 캐시
 - elasticsearch가 색인 문서들에서 필드의 값을 저장하는 인 메모리 캐시로 elasticsearch에서 무언가의 숫자를 세는 데 사용한다.
 - 데이터가 필요로 한 시점에 만들어지고, 다양한 작업 동안 유지된다.
 - 메모리에 올리는 작업은 많은 시간과 CPU를 소모한다.
 - warmers: elasticsearch가 내부 캐시를 채우기 위해 자동으로 동작하는 질의.

#### 사용처
 - 필드정렬, 집계시
 - 스크립트에서 doc['fieldname'] 표기법으로 필드의 값에 접근할때
 - _score 질의에서 특정 함수 사용시
 - 등..

#### 필드 데이터 관리
 - elasticsearch클러스터에서 JVM GC 이슈를 회피하는것.

 1. 필드 데이터에 사용할 메모리 제한
 - 특정 크기로 필드 데이터 공간을 제약한다.
 - elasticsearch.yml 파일에서 지정할 수 있음.

 2. 서킷 브레이커에 필드 데이터 사용하기
 - 처리되지 않은 요청에 의해 서버가 다운되지 않는도록 얼마큼의 메모리가 필요한지 예측하고 최대 사이즈를 넘어서는지 확인한다. 최대 크기를 넘어서면 Error
 - 노드가 동작중일 때 동적으로 조절이 가능하다.
 - 서킷브레이커는 기본적으로 JVM 힙사이즈의 60%로 필드 데이터 크기를 지정하고있다.
 - _cluster/settings/{ "transient": { "indices.breaker.fielddata.limit" } } 항목으로 지정

 3. doc values로 메모리값을 무시
 - 메모리에 적재된 문서를 색인하는 대신에 색인 데이터와 함께 디스크에 저장하고 그 값을 사용한다.
 - 메모리에서 데이터를 읽을 때 메모리 대신 디스크에서 읽는다.
 - 색인시점에 생성됨. 특정 필드에 "doc_value" : true|false 로 설정
