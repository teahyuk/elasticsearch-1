### Analyzer
- 색인을 할때 데이터는 검색어를 추출하기 위해 어떠한 프로세스를 거치는데 이를 분석이라 하며 분석을 위해 사용되는 것을 
   Analyzer라고 한다.
- Analyzer는 Tokenizer(1개) +토큰 필터(0개 이상)+ character filters(0개 이상) 구성된다.
- 분석은 색인 작업 뿐만 아니라 검색 작업에서도 일어난다.
- 분석기는 elasticsearch에서 제공하는 몇몇가지의 분석기가 있고 인덱스마다 Custom Analyzer를 만들 수 있다.
### Tokenizer
- 데이터를 설정된 기준에 따라 검색어 토큰으로 분리하는 작업을 한다.
- 각 토큰이 나타나는 시작 및 끝 offset을 저장한다.

### Token filter
- 토크나이저로 분리된 토큰들에 다시 필터를 적용해 실제 사용될 검색어들로 최종 변환하는 작업을 한다.
- 분석단계에서 가장 중요하다`

### Standard Analyzer
-    Unicode Text Segmentation 알고리즘에 정의 된데로 단어를 term 단위로 나눈다.
-  default 분석기
- Tokenizer
    - Standard Tokenizer
- Token Filters
    - Lower Case Token Filter
    - Stop Token Filter
- 옵션
    - max_token_length
    - stopwords
        - 불용어 등록
    - stopword_path
        - 불용어로 등록된 단어 파일 위치

### simple Analyzer
- 문자가 아닌 문자를 만나면 그 단위로 자른다 (" ", 2, - )
- TokenFileter는 없다.
- Tokenizer
    - Lower case Tokenizer

### whitespace Analyzer
- 공백 문자를 기준으로 token 생성
- TokenFilter은 없다.
- Whitespace Toeknzier 사용

### stop Analyzer
- simple이랑 유사한데 stop word(불용어)를 제거한다.
- Tokenizer
    - Lower case Tokenizer
- TokenFilter
    - Stop Token Filter

### keyword Analyzer
- 그냥 문자열 자체를 하나의 Token으로 하는것

### Pattern Analyzer
- 정규식을 사용하여 토큰을 분할하는 분석기
- java 정규식 표현을 사용
- Tokenizer
    - Pattern Tokenizer
- TokenFilters
    - Lower Case Token Filter
    - Stop Token Filter

### Language Analyzer
- 타입 필드에 언어를 명시하면 설정된 언어에 해당하는 분석기를 사용한다.

#### Tokenizer 및 token filter는 종류가 다양하다 . 즉 자신에게 맞는걸 가져다 쓰면 된다.


### 한글 형태소 분석기
-   은전 한닢
    - 자바 , 스칼라 인터페이스 제공
    - 복합 명사를 처리하기 위해서는 사용자 직접 등록 할 수 있는 사용자 사전 사용
    - 사용자 사전은 term 과 가중치의 형태로 구성
-  Nori 
    - 루씬에서 공식적으로 지원
    - nori _tokenizer , nori_part_of_speech(토큰 필터) , nori_readingform(토큰 필터) 으로 구성
    - 복삽 명사를 처리하기 위햔 옵션 decompund_mode , user_dictionary(사용자 사전)
    - nori_part_of_speech 품사태그 삭제
    - nori_readingform은 한자를 한글로 변경하는 역할
- 트위터 형태소 분석기
    - 현재 오픈소스로 개발되고 있음
    - git hub 확인 결과 거의 개발안됨 마지막 커밋이 1년전..ㄷㄷ
    
### 검색 결과 하이라이트
- 쉽게 말해 검색어를 포함한 다큐먼트가 있다면 이를 검색하고 검색어에 해당하는 부분에 강조 처리를 할 수 있다.


### 스크립트
- 스크립트를 이용해 사용자가 특정 로직을 삽입하는것을 스크립팅이라고 한다.
- 스크립트 전용언어 painless 사용
- 사용방식
    -  config 폴더에 스크립팅 저장
    - api 호출할 때 코드 내에서 스크립트르 직접 정의해서 사용
- rest api를 사용하여 script를 저장하고 사용할 수 있다.
    - 이때 저장되고 사용할때는 파라미터를 넘겨서 사용한다.
- 기본적으로 캐쉬 만료 시간은 없으나 설정 가능하다.   


### 검색 템플릿
- mustache 사용
- 간단한 예제 코드 
```
{
    "source" : {
      "query": { "match" : { "{{my_field}}" : "{{my_value}}" } },
      "size" : "{{my_size}}"
    },
    "params" : {
        "my_field" : "message",
        "my_value" : "some message",
        "my_size" : 5
    }
}
```
- json 형태로도 가능
```
{
    "source": "{\"query\":{\"bool\":{\"must\": {{#toJson}}clauses{{/toJson}} }}}",
    "params": {
        "clauses": [
            { "term": { "user" : "foo" } },
            { "term": { "user" : "bar" } }
        ]
   }
}
```
- 실무에서 검색 쿼리를 변경하는 일이 자주 발생한다. 알맞은 검색 템플릿을 제공하면 유연하게 대처 가능
- mustache 문법을 잘알아야 잘 사용 가능하다
- multi search template가능

### 별칭
- 실서버에서 인덱스에 오류가 발생하거나 매핑정보가 변경된다면 문제가 크다  인덱스를 재설정 할 때
까지 지속적인 오류가 발생한다.
- 이를 방지하기 위해 alias가 기능을 제공하며 멀티테넌시도 쉽게 가능하다.
- 즉 인데스가 추가되고 삭제되어도 클라 입장에서는 동일한 인덱스에 요청을 보낸다고 생각할 것이다.
- filter을 통해 원하는 필드 값만을 걸러내서 별칭을 만들 수 도 있다.
- 여러개의 인덱스로 하나의 별칭을 만들 수 있는데 is_write_index true라는 옵션을 인덱스에 주면 해당 인덱스에만 쓰기 작업이 가능
    - 실험결과 하나만 인덱스 중 하나만 가능하다
    - 모두 false이면 write는 안된다.
```
{
    "actions" : [
        {
            "add" : {
                 "index" : "test",
                 "alias" : "alias1",
                 "is_write_index" : true
            }
        },
        {
            "add" : {
                 "index" : "test2",
                 "alias" : "alias1"
            }
        }
    ]
}
```

### 스냅숏을 이용한 백업과 복구
- 백업은 언제나 중요하다
- 재색인을 통해 백업용 인덱스를 생성 할 수 있지만 데이터가 몇억건 이상이면 엄청 오래걸린다.
- snapshot api를 사용하면 클러스터 전체 및 개별 인덱스를 백업할 수 있다.
- 주의점은 버전간 호환성이 있다.
- snapshot을 저장하기 위해서는 먼저 레포지토리를 생성해야한다.
    - 만약 여러 클러스터가 동일한 레포지토리를 사용하고 있다면 하나의 클러스터만 쓰기 권한을 줘야 한다.
- (정확하지 않음) 동일한 인덱스를 snapshot 하면 엘라스틱 서치가 이를 확인하고 변경사항만 다시 snapshot을 저장한다.
- snapshot 이름은 항상 동일 리포지토리에서 유일해야 한다.
- snapshot 은 restore api를 사용하여 다시 복구 한다.

### Suggest API
- 검색어의 텀이 정확히 일치 하지 않아도 자동으로 인식해서 처리 할 수 있도록 하는 API
- Term Suggest 추천단어 제안
    - 편집거리 계산 알고리즘 사용
    - 즉 비슷한 문자를 결과값으로 돌려주면 얼마나 유사한지 score 를 제공한다.
- Completion Suggest 자동완성 제안
    - 사용하기 위해서는 매핑시 suggest 타입으로 completion 을 설정해야한다.
    ```
    {
        "mappings": {
            "properties" : {
                "suggest" : {
                    "type" : "completion"
                },
                "title" : {
                    "type": "keyword"
                }
            }
        }
    }
    ```
    ```
    {
      "suggest" : [ "Nevermind", "Nirvana" ]
    }
    ```
    ```
    {
        "suggest": {
            "song-suggest" : {
                "prefix" : "nir", 
                "completion" : { 
                    "field" : "suggest" 
                }
            }
        }
    }
    ```
    - prefix를 통해 앞글자로 시작하는 자동완성 결과를 가져온다
    - 부분일치를 하기 위해서는 기존 input에 배열로 넣어야한다.
    - fuzzy , regex 모두 가능
    
- Phrase Suggest 추천 문장 제안
    - 인덱스를 만들때 먼저 Phrase suggest를 위한 매핑 작업이 필요하다
    - https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-phrase.html
    
- Context suggest 추천 문맥 제안
    - https://www.elastic.co/guide/en/elasticsearch/reference/current/suggester-context.html

- boosting query
    - positive에 일치하는 문서를 반환하고 negative에 해당하는 문서의 score을 줄이는 쿼리
    - 필수 옵션 positive , negative , negative_boost
    - positive에 해당하는 결과와 negative에 해당하는 결과가 같으면 score 에 negative_boost를 곱한다.
    
### 몰랐던것
- 샤드의 상황에 따라 같은 유형의 단어여도 score가 다르다.
    - 그 이유는 기본적으로 bm25를 사용하는데 전체 document수에 영향을 받기 때문에 score가 차이가 날 수 있다.
    - 해결 방법?
        - 인데스에 문서를 많이 넣어라
        - ?search_type=dfs_query_then_fetch 사용
            - 모든 샤드로 부터 가져온 후 score계산을 한다.
            
-   function score
    -   질의어에 해당하는 다큐먼트의 score을 변경 가능하도록 하는 것
    -   score_mode -> fuctions 안에 정의된 쿼리 스코어를 어떻게 계산 할 것이냐(각각 필드에 저장된 weight)
    -   boost_mode ->  function query와 main query score을 어떻게 계산 할 것인가 
    - script , random , weight 등 다양하게 score에 영향을 줄 수 있다.
    -   field Value factor
        - script랑 마찬가지로 문서의 필드를 사용하여 score에 옇향을 줄 수 있지만 오베헤드가 적다
            -   script가 오버헤드가 큰 이유는 아마 compile 때문일 것 같다
        ```
        {
            "query": {
                "function_score": {
                    "field_value_factor": {
                        "field": "likes",
                        "factor": 1.2,  // score에 영향을 줄 점수
                        "modifier": "sqrt", // 게산하고자 하는 방식
                        "missing": 1 // 해당 field가 없는 경우 부여 되는 점수
                    }
                }
            }
        }
        ```
        -   field_value_score  음수가 되면 안된다 그런데 log는 음수 및 에러를 발생 시키기에 log1p , log2p사용
        -   decay
            - 사용자가 주어진 다큐먼트에 numeric 필드,geo-point의 거리에 따라 점수를 감소 시킨다.
            ```
            "DECAY_FUNCTION": { 
                "FIELD_NAME": { 
                      "origin": "11, 12", // 사용자 주어진 점수
                      "scale": "2km", 
                      "offset": "0km",
                      "decay": 0.33 // 감소 비율
                }
            }
            ```
            -   oring - offset-scale 이 부터 decay가 적용된다.
- search -as -you - type
    - 인덱싱 된 값에 부분일치를 효과적으로 하기 위해 하위 필드를 두는 타입
    - shingle token filter을 통해 my.field,my.field._2gram,my.field_3gram 생성
    -   my_field._index_prefix는 my.field_3gram 을 다시 한번 edge ngram token filter로 wrapping

- rank_feature
    -   숫자 기능 벡터를 색인 할 수 있다.
    - 솔직히 잘 모르겟다....
    - reank_feature 쿼리는 rank_feature 타입에만 사용 할 수 있다.
        - 해당 쿼리는 숫자 기능의 가치를 기반으로 점수를 높인다.
        - 비경쟁 히트를 효육적으로 건너 뛸수있는 이점이 있다.
        
- 서킷 브레이커
    -   서킷 브레이커라는 패턴도 존재
        - 쉽게 말하면 서비스 A를 통해 서비스 B로 가는 로직이 있다.
        - 서킷브레이커는 이 두 서비스 가운데에 있어 B 서비스의 오류가 발생 시 이를 인지하고 서비스 A로 부터 넘어오는 모든 호출을 막아버린다.
        -   https://bcho.tistory.com/1247
    -   엘라스틱 서치에서의 서킷 브레이커
        - 작업하는 동안 OutOfMemoryError가 발생 하지 않도록 하는 차단기 역할
        - 차단기는 여러개가 있으며 각 차단기 마다 메모리 사용 제한 이 있으며 부모 레벨의 차단기는 총 메모리 사용 제한을 가지고 있다.
    - 부모 서킷 브레이커
    -   fielddata-circuit-breaker
        -   필드를 메모리에 로드 할 수 있는 메모리 양을 관리 기본은 jvm heap memory 40%
    -   request-circuit-breaker
        -    요청당 집계 계산에 사용되는 메모리
    -    in-flight-circuit-breaker
        -   전송 또는 HTTP에서 현재 활성 상태인 모든 수신 요청의 메모리 사용량을 제한
    -   accounting-circuit-breaker
        - 완료된 요청에 대한 해제 되지 않은 메모리에 보관된 항목의 메모리 제한
    -   script-compilation-circuit-breaker
        -  스크립트 컴파일 횟수 제한

- 필드 데이터 캐시
    -   field를 정렬 할때에는 역인덱스를 사용하는 것이아니라 정렬 기준이 되는 field를 위해 document에 접근해야 한다.
    -    즉 정렬시 term을 역인덱스에 매핑 시키는 것이 아니라 document를 term에 mapping 시켜야 하는데 이를 uninverted index라고 한다.
    -   uninverted index를 활용하기 위해 elasticsearh는 내부적으로 query에 해당하는 document에 모든 필드를 메모리에 적재한다.
    - 필드 데이터 캐시를 사용하는 것에는
        -   Sorting on a field
        -   Aggregations on a field
        -   Scripts that refer to fields
     - 필드 데이터 캐시로 인해 oom , CircuitBreakingExcpetion이 일어 날 수 있다.
    -   그래서 사용하는게 elasticsearch 서킷 브레이커