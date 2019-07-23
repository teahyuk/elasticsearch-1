### 검색 API
-   검색을 하게 되면 하나의 샤드에서 검색이 되는게 아니라 빠른 속도를 위해 여러 샤드로 부터 검색에 대한 응답을 취합한다.
	
-   만약 하나의 샤드에서 오류가 발생 하면 coordinating node가 복사본 샤드에 해당 요청을 보낸다.
    -   여기서 궁금증 ? 
		- 모든 샤드는 fail over을 위해 복제본을 가지고 있다.
		- 그럼 검색시 중복이 발생하지 않을까? 그 이유는 복제본과 primary shard에서 검색을 할 꺼니깐?
		- 엘라스틱 서치는 내부적으로 라운드 로빈 , 동적 분배 방식의 알고리즘을 제공하기에 검색 시점에 알맞은 샤드를 선택해 준다.
- Multi search
	- 서로 다른 인덱스를 동시에 검색 가능
	
- 검색시 timed_out설정을 해줘야한다
    - 기본 값은 무제한 이기 때문에 영영 안올 수있다.
    
-   검색을 하는 방식
	- url query 방식
		- 간단하다
		- 간단함 뿐이다 복잡한 쿼리를 사용하기에는 가독성이 떨어지면 모든 옵션을 사용 할 수 없다.
			
	- request body 형식
		- json을 모르면 가독성이 떨어진다 , 간단하지 않다.
		- json에 익숙하다면 가독성이 높고  다양한 옵션을 지원한다.
		- json형식에 query dsl을 사용  dsl이란 domain specific language를 의미
		- search_type, request_cache , allow_partial_search_results는 쿼리 파라미터로 설정하며 나머지
		  옵션은 body안으로
			
- 다양한 옵션이 존재한다 많이 연습하고 익숙해지는게 좋다.	

	
- 페이징
	- 원하는 개수 만큼 결과를 가져 올 수 있다.
	- from/size 키워드를 사용
		- from+size에도 개수 제한이 있다.
		- index.max_result_window보다 크면 오류 발생
	- 단점인가? rdb와는 다르개 개수 만큼만 읽는게 아니라 데이터 전체를 다읽고 사이즈만큼 필터를 한다.

- 정렬 
    - 각종 옵션 존재
    - 해당 필드가 존재하지 않으면 검색이 안된다. 이를 무시하고 검색할 수 있는 옵션 존재
        - unmapped_type : long
            엘라스틱 서치가 내부적으로 long 유형이 있는것 처럼 매핑한다.

- _soucre 필터링
    - 원하는 필드만 제공하거나 , 전부 제공을 안 할수 있다.

- search type
    -  query_then_fetch(default 값)
        - 전체 샤드의 검색이 다 수행된 후에 결과를 출력
    
    -   dfs_query_then_fetch
            -   query_then_fetch와 동일하며 스코어링을 위해 전체 도큐먼트의 검색어 빈도수를 계산
    
    // 이 두개도 공식 문서 상에는 없다...
    -   query_and_fetch
        -   샤드별로 검색 되는 대로 결과를 받아 출력한다.
       
    -   dfs_query_and_fetch
        -   검색 방식은 query_and_fetch랑 동일하며 스코어링을 위해 전체 도큐먼트의 검색어 빈도수 계산
    
    // curl 쿼리 던져본 결과 No search type for ~~ 둘다 remove 
    -   count
        -   검색된 도큐먼트 정보를 배제하고 전체 hits수만 출력

    -   scan
        -   scroll과 같이 사용되며 검색 결과를 바로 보여주지않고 scroll_id를 사용해서 나중에 결과를 출력
        
- range 검색
    - 날짜 , 숫자 타입 검색의 경우 범위 기준 검색이 가능하다.
    - lt(<),gt(>),lte(<=),get(>=)

- operator
    - 엘라스틱 서치는 검색시 문장이 들어올 경우 or 연산자를 한다.(default)
        - hello world라는 검색어가 들어오면 hello, world 둘 중하나를 포함한 document를 검색한다.
    - operation 연산을 통해 명시적으로 and를 사용 할 수 있다.
    - minimum_should_match
        - 맞아야 하는 텀의 수
        - hello world tree 라는 검색어가 들어오고 해당 설정이 2라면 hello,world,tree 중 최소 2개 이상 맞는
           document만 검색이 된다.

- fuzzy
    - fuzzy 쿼리도 존재하며 option으로도 사용
    -   옵션 fuzziness 
    - value 0,1,2,3,auto
    - 해당 value에 맞게 오차가 있는 document도 찾는다
    - 검색어 Hio 이고 fuzziness value 1 이면 Hiot document도 찾아진다. 
 

- Query dsl은 두가지의 형태로 나뉜다
	- 쿼리 컨텍스트
		- 전문 검색시 사용 즉 분석기에 의한 분석이 필요로 하고 루씬 레벨에서 분석을 하기에 조금 느리다
		- 문서와 쿼리와 얼마나 유사한지 score 검사
		- 캐싱되지 않고 디스크 연산을 수행하기 느리다
	- 필터 컨텍스트
		- 조건 검색이다 엘라스틱서치 레벨에서 처리가 가능하기에 상대적으로 빠르다.
		- 스코어를 계산하지 않는다
		- 자주 사용되는 필터의 결과는 엘라스틱 서치가 내부적으로 캐싱
			- 캐싱 방식은 LRU
			
	- 두개의 가장 큰차이점은 score부분에 있다 . _score는 검색의 정확도를 의미한다. filter는 score계산하지 않는다.
		- score를 계산 하는 알고리즘이 있다
		- 어떻게 score를 계산하는지 보고 싶다면 쿼리 마지막에 "explain":"true"

- term 쿼리
    - 검색어가 하나의 토큰이므로 해당 토큰과 일치하는 document 만 search
    ```
       "term":"seungki"
   ```
    - terms로 여러개 검색어 가능
   ````
    "terms":["lee","seungki"]
    
 
- match 쿼리
    - Full text queries
        - match 쿼리 말고도 다양하게 존재
    - 검색어가 Analyzer에 의해 토큰 형태로 나누어진다
    - 검색어 the prince  --> "the","prince" 의 형태로 나누어져서 둘중 하나라도 포함하는 document search
    - multi match 도 가능
        - query를 여러 필드에 적용 하는 방법

- 불 쿼리
    -   compound query의 일종
        - 하나의 쿼리나 여러 개의 쿼리를 조합해서 더 높은 스코어를 가진 쿼리 조건으로 검색 수행
        - 조건 쿼리문
        - must 
            - 반드시 해당
        - must not
            - 반드시 해당 되면 안된다
        - should
            - 해당 될 필요는 없지만 해당이 된다면 높은 스코어를 갖는다.
            
### 몰랐던 내용들
- scroll
    - 페이지를 통해 많은 양을 가져오는건 메모리상에 낭비이다. 그래서 scroll이라는 개념이 있는데
    - 처음 요청시 size 옵션을 가진 query 문을 날린다.
    - 그 결과 이에 해당하는 document랑 scroll id가 날라온다
    - 그 scroll id를 사용해서 쿼리를 날리면 그다음 결과를 가져온다. scroll은 메모리상에 저장이 되는데 이는
       일정 시간동안만 존재한다.
		
- BM25
    - 검색에 대한 score 계산하는 algorithm
    - idf + tfnorm
         - idf 문서에서 자주 등장하는 단어 일 수록 낮은 가중치를 준다는 공식
         
    - tfnorm
        - 문서내에서 같은 단어가 여러번 등장한다면 그 단어에 높은 가중치를 주는 방법

- match phrase query
    - 해당 쿼리는 term의 순서를 보장한다.
    - 보통 검색시 term의 순서를 보장하지 않기에 elastic search , search elastic 은 같은 결과다
    - 그러나 해당 쿼리문은 elastic search 의 순서를 보장한다.
    - slop 옵션에 따라 elastic 과 search 사이에 허용가능한 문자수를 의미한다.
    
 - term , terms
    - 해당 쿼리는 keyword 필드에 안성 맞춤이다
    - term은 text에도 사용할 수 있지만 권장하지 않는다
    https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html#term-top-level-params

- inner hit
    - elasticsearch 6.x 버전부터 타입이 하나이기 때문에 parent/child 구조가 사라지고 이를 대신할 join data type이 생겼다.
        - join data type은 field간에 관계를 설정해주는 것이다.
        - 해당 옵션을 사용하여 child , parent를 저장할 때에는 같은 샤드에 보관 되어야 하기에 routing 정보도 같이 저장
    - 기존에는 has_child_query 를 사용시 parent의 document만 나왔기에 child를 위해 추가질의가 필요
    - inner_hits 파라미터를 통해 child도 한번에 검색이 가능하다.
    - 