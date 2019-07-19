## 데이터 검색_Day1

### 데이터 검색 기본
 - GET 메서드를 이용하여 데이터를 검색한다.
 ~~~
 -X{method} http://{host}:{port}/{index}/{type}/{documentid}
 ~~~
 - 어디를? : 특정 index의 특정type을 보게끔 지시할수 있지만 같은 index, 다수의 index, 혹은 모든 idex에서 여러 타입에 검색 할 수도 있다.
    - N개의 타입 검색
    ~~~
    -XGET http://{host}:{port}/{index}/{type1, type2, ..}/_search ...
    ~~~
    - 특정 index의 모든 type에 검색
    ~~~
    -XGET http://{host}:{port}/{index}/_search ...
    ~~~
    - N개의 index 검색
    ~~~
    -XGET http://{host}:{port}/{index1, index2}/_search ...
    ~~~
    - 모든 index 에서 검색
    ~~~
    -XGET http://{host}:{port}/_search ...
    -XGET http://{host}:{port}/_all/{type}/_search ...
    ~~~

 - 응답내용
    - 검색응답은 검색 기준과 일치하는 문서뿐 아니라 검색 성능이나 결과의 유사도를 확인하는 데 유용한 정보를 포함한다.
    ![응답내용](../../img/데이터검색/데이터검색_0.jpg)
    - 만약 3개의 노드를 가지고 있는 클러스터가 있고, 각각은 하나의 샤드가 있고 복제가 없다. 하나의 노드가 내려가면, 어떤 데이터는 놓칠 수 있다.
    -> elasticsearch가 살아있는 샤드로부터 결과를 주고, 실패한 필드에 검색하기 위해 이용 불가능한 샤드 수를 보고한다.

 - 무엇을 어떻게?
    - 쿼리로 질의하기_query_string
    elasticsearch는 JSON 포맷으로 모든 검색 기준을 명시하도록 해준다.
    ~~~
    {uri} -d '
      {
        "query": {
          "query_string": {
            "query" : "elasticsearch",
            ... {query_string 옵션들}
          }
        }
      }
    '
    ~~~
    - term 쿼리
