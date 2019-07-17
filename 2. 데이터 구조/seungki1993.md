### 인덱스 생성
  - curl -XPUT localhost:9200/top -d

### Doucument 저장
  - PUT ,POST 메소드를 사용해 Document를 저장
  - PUT을 사용하여 Document를 저장시 직접적으로 id를 기재
     -  PUT 메소드를 통해 같은 인덱스 , 타입 , 아이디를 입력한다면 기존 데이터가 삭제되고 다시 Insert된다.
  - POST에 경우에는 임의 아이디가 부여 된다.
 
 

### Dcoument 저장 및 검색
```
"_index" : "books",
  "_type" : "book",
  "_id" : "1",
  "_version" : 1,
  "result" : "created"
 ```
  
- 해당 정보는 ctx 정보이다. 
    - ctx란 업데이트 할 개체의 원본(_source)에 액세스 할 수있는 특수 변수

### Document 및 인덱스  삭제
- Document 삭제시 메타 정보는 그대로 남아 있다.
    - 해당 document를 삭제하고 동일한 document를 다시 저장한다면 version은  1부터가 아니라 그전 삭제된 document의 버전을 그대로 이어 받는다.
    - 무조건이 아니라 index.gc_deletes 시간 안에서만 가능하다
- 인덱스 삭제 시 모든 메타 정보까지 다 삭제된다
    - 즉 document처럼 데이터가 그대로 남아 있지 않는다.    

### Document 업데이트
   - doc , script를 이용해서 데이터를 제어 할 수 있다.
  -  doc은 단순하게 필드를 추가하거나 기존 필드를 변경하는데 사용한다.
  - script는 복잡한 프로그래밍 기법을 사용해서 입력된 내용에 따란 필드의 값을 변경하는데 사용한다.
  -  doc
        - PUT test/_doc/1 
      ````
            "counter" : 1,
            "tags" : ["red"]
        ````
   - script 
      -  POST test/_update/1 <br/>
      ````
            "script" : 
                "source": "ctx._source.counter += params.count",
                "lang": "painless",
                "params" : 
                    "count" : 4
     ````
- update는 도큐먼트의 구조를 변경하는 것이 아니라 실제로 저장된 도큐멘트 값을 가져와서 명령을 토대로
새로운 다큐먼트를 만들고 새로 만들어 진것을 기존 다큐먼트에 입력하는 방식이다.

- script는 MVEL 언어 문법을 사용하는데 자바 런타임 플랫폼 위에서 동작하기 위해 만들어진 언어 이므로 자바 문법과 유사하게 동작
    - 5.0 부터는 ElasticSearch에서 직접 개발한 Painless 스크립트를 사용
    
- script에서는 if문등 다양하게 사용할 수 있다.(== , != =<) 또한 ctx.op를 통해 해당 document를 삭제 할 수 있으며
  document에 field도 삭제 할 수 있다.
  
- detect_noop
    - 해당 설정을 true를 보내면 변경사항이 있는지 먼저 확인하고 없다면 무시하는 설정이다.
    - 해당 설정이 있는 이유는 doc을 사용해 변경을 하게 되면 아무런 변화가 없어도 update를 실행하기에 낭비다.
    
- Upserts
    - upserts 속성은 다큐먼트를 수정하고자 할때 해당 다큐머트가 없으면 upsert에 설정으로 새로운 다큐먼트를 만든다.


### Document의 version
-   해당 document에 변경된 횟수를 알 수 있다.
- version 정보는 알 수 있지만 다시 되돌리지는 못한다.

### 파일을 사용한 Document 처리
-    curl -XPUT localhost:9200/books/book/1 -H 'Content-Type:application/json' -d @{파일이름}

### Bulk
-    데이터에 대한 많은 처리를 하나의 api call로 처리 할 수 있는 방법
- 당연 하나씩 하는것 보다는 빠르다
- 주의점은 각행의 마지막에는 \n 있어야 한다.
- 벌크 API에서는 index , create, delete ,update 4가지 동작을 처리 가능
    - 주의점 index , create , update동작은 실행 메타정보 , 요청데이터를 같이 보낸다. delete는 실행 메타 정보만 필요
    - 실행 메타 정보 { "index" : { "_index" : "test", "_id" : "1" } }
    - 요청 메타 정보 { "field1" : "value1" }
    - 주의점 이미 있는 document를 create 하면 오류 발생
 - bulk로 업데이트시 retry_on_conflict속성을 사용할 수 있는데 해당 속성은 version이 충돌할 경우 재시도 할 횟수이다.
 - 또한 bulk에서도 script , doc 등을 사용할 수 있으며 파일로도 bulk 작업 수행 가능하다.


### 매핑
-   데이터의 저장 형태와 검색 엔진에서 해당 데이터에 어떻게 접근하고 처리하는 지에 대한 명세
- RDB 스키마라고 생각하자
- 설정 방법에는 
    -  인덱스 생성하면서 매핑을 설정
    -  매핑 API를 통해서 설정 가능 _mapping
- mapping type은 Meta-fields , fileds or propertires 
- 매핑이 설정 되어 있지 않은 상태에서 Document를 저장하면 자동으로 mapping이 설정이 된다.
- 데이터 타입은 properties로 한다.
- 매핑 옵션 중 index 옵션이 있는데 해당 옵션은 색인을 할지에 대한 옵션인데 만약 false이면 데이터가 source에는
  저장이 되지만 결과적으로 검색은 할 수 없다.



### 매핑 _soucre
-   source의 옵션을 enabled하면 원본 데이터는 저장이 되지 않고 색인된 토큰들만 저장이 된다.
- 즉 쿼리로 검색시 검색은 되나 원문 데이터는 볼 수 없다.
- exclude,include

### 문자열 타입
- 기존 String으로 했지만 현재는 text , keyword로 나누어진다.
- 두개의 큰차이점은 문자열을 값을 전체로 인식 하거나(keyword) , 문자열이 analyzer을 거치냐 차이(text)
- 각종 option 설명은 document에 친절하게 영어로 설명이 되어있다.

### 숫자 타입
- 검색어로 검색이 불가능 하지만 주어진 기준값과 비교는 가능하다(> => <= <)
- 숫작타입이지만 문자열이 들어왔을때 강제로 저장하는 옵션이있다

### 날짜 타입
- 엘라스틱 서치 엔진 내부적으로 long 타입의 숫자로 저장

### Object 타입
- dynamic이 가능하다 . 이말은 기존 매핑에 없던 filed가 들어와도 설정에 따라 동적으로 추가 될 수 있다. 
    - 해당 설정값이 false여도 매핑에 없는 값이 저장 될 수 있으나 해당 컬럼은 색인이 되지 않아 검색을 할 수 없다.
    
### 중첩 매핑
- 내가 이해하기로는 객체 필드와 유사하지만 객체 필드안에 배열처럼 여러 값이 들어가는것같다...
```aidl
 "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
```
- 중첩 필드는 각각의 독립된 데이터로 저장이 되기 때문에 nested 질의를 사용해야한다.

### 좌표를 위한 geo point

### 위치 모형을 위한 get shape

### 다중 필드
- 에를 들어서 한 타입이 기본 타입이 text이다 . text는 anlyzer을 통해 토큰 형태로 인댁싱이 된다. 그렇게 되면 
   나누어진 토큰으로 검색이 가능하다. 그러나 해당 필드에 대해 keyword처럼 전체 문장으로만 검색 하고 싶다면
   다중 필드를 사용하면 된다. 그럼 토큰 형태로도 검색이 가능하고 문장 전체로도 검색이 가능하다.
   
### 필드 복사
- copy_to 옵션을 통해 해당 필드를 다른 필드에 복사 할 수 있다.

