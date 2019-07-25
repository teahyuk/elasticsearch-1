### 집계
- facets는 remove되었다.
- faccets 대신 aggreagtion을 사용

### Aggregations
-   검색 쿼리를 기반으로 집계 된 데이터를 제공
- 자주 사용되는 aggregation은 캐시를 해서 사용 가능하다.(웹사이트 홈페이지에 사용되는 어그리게이션)
- Bucketing , Metric, Matrix , Pipeline Aggregations 존재

### Bucket Aggregations
- 주어진 조건에 해당하는 도큐먼트를 버킷이라는 저장소 단위로 구분해서 담아 새로운 데이터 집합을 형성
    - bucket 이란?
        - 특정 기준을 만족하는 document의 모음이다.
- 버킷별로 하위 Aggregations을 통해 버킷에 있는 데이터로 다시 새로운 Aggregations 연산을 반복해서 수행 가능
- 레벨이 깊어 질수록 서버 메모리와 같은 자원의 소비가 많이 늘어난다.
- metrics 계산은 하지 않는다.
- 단일 응답에 대한 최대 bucket 수는 10000 default
- bucket aggreations 의 종류
    - https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket.html 다양하다
#### 글로벌 애그리게이션
- 조건에 의해 걸러진 document가 아닌 인덱스의 전체 document 를 하나의 bucket 으로 만든다.
#### 필터 , 누락 애그리게이션
- filter에 걸러진 document가 하나의 버킷으로 만들어 진다.
    - filters를 사용해 멀티 bucket 가능
        ```
             "aggs" : {
                 "messages" : {
                   "filters" : {
                     "other_bucket_key": "other_messages", // 그이외에 모든 document에 해당하는 bucket
                     "filters" : {
                       "errors" :   { "match" : { "body" : "error"   }},// error에 해당하는 bucket
                       "warnings" : { "match" : { "body" : "warning" }} // warning에 해당하는 bucket
                     }
                   }
                 }
               }   
        ```
- 누락은 해당 필드가 없는 document를 하나의 bucket으로 만든다.
#### Term 애그리게이션
-   검색된 텀별로 버킷을 생성한다.
- type이 text이면 매핑시 fielddata 속성을 줘야 한다.
- size 옵션은 결과를 출력하고 싶은 버킷의 수를 의미한다.
    - document 카운트는 완전히 정확하지 않다.
    - 그렇기에 정확한 카운트를 하고 싶으면 size값을 늘리면 되는데 이는 메모리에 부담이 된다.
    - 그래서 이를 해결할 방법이 shard_size를 이용하는 것인데... 잘 이해가 가지 않는다.
-   정렬 옵션
    -   _count 버킷에 담긴 도큐먼트 개수
    -   _term 버킷을 구분하는 텀 값의 이름
- min_doc_count 옵션을 주게 되면 bucket이 가져야 할 최소 document count를 설정 할 수 있다.
    - default 1
    - 모든 샤드에서 수집한 이 후 수행
    - 즉 이숫자에 해당하지 못한 document는 출력이 안된다.
- script 도 가능하다.
- collect mode 도 존재
    - 상위 수준의 aggs 완료 될때 까지 하위 집계연산을 지연시키는 것이 효과적이다.
    - value 로는 breadth_first ,depth_first (bfs , dfs)
- 결과 중 doc_count_error_upper_bound, sum_other_doc_count
    -    doc_count_error_upper_bound
        -   집계 수행 시 발생한 오류 개수
    -   sum_other_doc_count
        -   버킷에 포함되지 않은 총 도큐먼트 개수
#### 범위, 날짜 애그리게이션
-   기본적으로 Range 애그리게이션이 존재 
    -   두개의 차이점은 from,to에 date Math 표현이 가능하며 날짜 형식도 지정할 수 있다.
- to , from을 기본적으로 사용
- timezone 설정 가능
- keyed 옵션
    - 결과를 key: value에 형태로 제공
```
"aggs": {
        "range": {
            "date_range": {
                "field": "date",
                "format": "MM-yyyy",
                "ranges": [
                    { "to": "now-10M/M" }, 
                    { "from": "now-10M/M" } 
                ]
            }
        }
    }
```
#### 히스토 그램 애그리게이션
-    일정 간격으로 구분한 버킷을 생성 (numeric 만 가능)
-   간격을 계산 하는 공식 Math.floor((value - offset) / interval) * interval + offset
-   간격에 해당하는 document가 없으면 뛰어넘는다
    - min_doc_count설정을 0을 주면 출력가능
-   extended_bounds
    - min_doc_count 옵션과 유사하나 차이점은 min , max를 통해 시작점과 끝점을 정할 수 있다.

#### 날짜 히스토 그램 애그리게이션
-   히스토 그램과 유사하지만 해당 애그리게이션은 date value에만 적용 가능
- calendar intervals 인터벌 간격이 1
    -   minute, hours,day,week,month,quarter 
- fixed intervals
    -   milliseconds , seconds,minutes,hours,days
- timezone 가능
- script 사용가능

#### 위치 거리 애그리게이션
-   기준점을 중심으로 거리를 구간별로 구분해서 버킷으로 분리
-   unit
    - m , mi , in , yd, km , cm , mm
    ```
    "aggs" : {
            "rings_around_amsterdam" : {
                "geo_distance" : {
                    "field" : "location",
                    "origin" : "52.3760, 4.894",
                    "ranges" : [
                        { "to" : 100000 },
                        { "from" : 100000, "to" : 300000 },
                        { "from" : 300000 }
                    ]
                }
            }
        }
    ```

### 매트릭 어그리게이션
- 주어진 조건으로 도큐먼트를 계산해서 처리된 결과값을 도출한다.
-   필드에서 숫자 타입으로 동작하며 숫자 필드의 집계값을 계산한다.
- min , max, sum ,avg, stats(min,max,sum,avg를 다가져온다)
- cardinality
    - 특정 요소(필드)의 개수를 구하는 집계


### Pipeline Aggregations
-   다른 Aggregation에서 받은 결과값을 input으로 사용해 다시 리턴하는 집계
-   Parent , Sibling 타입이 존재
   - Parent 
        - 상위 집계에 대한 결과를 가지고 집계를 수행한다.
        ```
        "size": 0,
            "aggs" : {
                "sales_per_month" : {
                    "date_histogram" : {
                        "field" : "date",
                        "calendar_interval" : "month"
                    },
                    "aggs": {
                        "sales": {
                            "sum": {
                                "field": "price"
                            }
                        },
                        "sales_deriv": {
                            "derivative": {
                                "buckets_path": "sales" 
                            }
                        }
                    }
                }
            }
        ```
        
    - Sibling
        - 상위 집계가 아닌 동일 선상에 있는 집계에 대해 집계를 수행한다.
        ```
        "size": 0,
          "aggs": {
            "sales_per_month": {
              "date_histogram": {
                "field": "date",
                "calendar_interval": "month"
              },
              "aggs": {
                "sales": {
                  "sum": {
                    "field": "price"
                  }
                }
              }
            },
            "avg_monthly_sales": {
              "avg_bucket": {
                "buckets_path": "sales_per_month>sales" 
              }
            }
          }
        ```
            
-   buckets_path
    -   파이프라인 집계에 사용될 집계를 참조 할 때 사용하는 키워드

### Matrix 집계
-   여러 필드에서 작동하며 document 필드에서 추출한 값 기반으로 metrix작업 수행
-   elastic search document에 해당 집계는 삭제되거나 변경될 것이라는 warning이 있다.
    
