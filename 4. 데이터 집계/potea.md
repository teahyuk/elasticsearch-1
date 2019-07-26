# 엘라스틱서치 실무가이드

## 05 데이터 집계
- 집계
- 메트릭 집계
- 버킷 집계
- 파이프라인 집계
- 근삿값(Approximate)으로 제공되는 집계 연산

### 집계
- 데이터를 그룹화해서 각종 통계 지표를 제공
- X-Pack을 사용할 경우 Ansi SQL 구문으로 질의 가능
- Keyword 타입은 Text 타입과 달리 분석을 수행하지 않기 때문에 집계 성능 향상


#### 엘라스틱서치와 데이터 분석
- 엘라스틱서치는 데이터를 분산 관리하여 문서의 수가 늘어나도 배치 처리보다 좀 더 실시간에 가깝게 문서를 처리
- 집계를 여러개 중첩해서 사용 가능

#### 엘라스틱서치가 집계에 사용하는 기술
- 집계는 검색보다 더 많은 리소스를 사용
- 데이터의 양이 클수록 CPU와 메모리 자원 소모
- 같은 질의에 대해 버퍼(캐시)에 보관된 결과를 반환

##### 캐시
- Node query Cache
    - 노드의 모든 샤드가 공유하는 LRU(Least-Recently-Used) 캐시
- Shard request Cache
    - 샤드에서 수행된 쿼리의 결과를 캐싱
- Field data Cache
    - 집계 계산되는 동안 필드의 값을 메모리에 보관

#### Aggregation API

```
    {
        "query": { .... },
        "aggs": { .... }
    }
```

- 집계와 질의를 함께 사용하면 질의의 결과 영역 안에서 집계가 수행: 집계 영역(Aggregation Scope)
    - 질의가 생략되면 내부적으로 match_all 쿼리로 수행
    - 글로벌 버킷을 사용하면 질의 내에서도 전체 문서를 대상으로 집계 수행가능

```
    "aggregations" : { 
    "<aggregation_name>" : { 
        "<aggregation_type>" : { 
            <aggregation_body> 
        } 
        [,"aggregations" : { [<sub_aggregation>]+ } ]? 
    } 
    [,"<aggregation_name_2>" : { ... } ]* 
} 
```

1. 버킷(Bucket)
    - 버킷: 쿼리 결과로 도출된 도큐먼트 집합에 대해 **특정기준으로 나누어 놓은 도큐먼트들의 모음**
    - 버킷에 대한 산술 연산 수행
2. 메트릭(Metric)
    - 쿼리 결과로 도출된 도큐먼트 집합에서 필드의 값을 산술 연산 수행
3. 파이프라인(Pipeline)
    - 다른 집계 또는 관련 메트릭 연산의 결과를 집계
4. 행렬(Matrix)
    - 실험적인 기능
    - **버킷** 대상이 되는 도큐먼트의 여러 필드에서 추출한 값으로 행렬 연산 수행

### 메트릭 집계
- 단일 숫자 메트릭 집계(single-value numeric metrics aggregation)
    - 합산 집계
        - type: "sum"
        - script를 사용하여 단위 등을 적용
    - 평균 집계
        - type: "avg"
    - 최소값 집계
        - type: "min"
    - 최대값 집계
        - type: "max"
    - 개수 집계(Value Count)
        - type: "value_count"
- 다중 숫자 메트릭 집계(multi-value numeric metrics aggregation)
    - 통계 집계
        - 합, 평균, 최소값/최대값, 개수를 한번의 쿼리로 집계
        - type: "stats"
    - 확장 통계 집계
        - 통계 집계에 분산, 표준편차 등의 통계값이 추가
        - type: "extended_stats"
- 근사값(Approximate count)를 기반으로 한 결과 제공
    - 카디널리티 집계(Cardinality aggregation)
        - 단일 숫자 메트릭 집계
        - 개수 집합과 유사하게 횟수를 계산하지만 중복값을 제외한 고유값에 대한 집계 수행
        - 중복값을 제외하는 것은 성능에 영향을 주어 근사치를 통해 집계 수행
        - type: "cardinality"
        - [HyperLogLog++ 알고리즘](https://en.wikipedia.org/wiki/HyperLogLog)을 기반으로 동작
    - 백분위 수 집계(Percentiles aggregation)
        - 다중 숫자 메트릭 집계
        - [백분위 수](https://ko.wikipedia.org/wiki/%EB%B0%B1%EB%B6%84%EC%9C%84%EC%88%98)
        - type: "percentiles"
        - TDigest 알고리즘
    - 백분위 수 랭크 집계(Percentile Ranks aggregation)
        - 다중 숫자 메트릭 집계
        - type: "percentile_ranks"
- 지형
    - 지형 경계 집계
        - 지형 좌표를 포함하고 있는 필드에 대해 해당 지역 경계 상자를 계산
        - 필드 타입이 geo_point여야 사용 가능
        - 가장 끝부분에 위치한 정보로 경계가 정해진다.
        - type: "geo_bounds"
    - 지형 중심 집계
        - 지형 경계 집계의 범위에서 정 가운데의 위치 반환
        - type: "geo_centroid"

### 버킷 집계
- 메트릭 집계와는 다르게 계산하지 않고 버킷을 생성하여 중첩된 집계를 가능하게 한다.
- 당연히 중첩될 수록 메모리 사용량이 증가
- 범위 집계
    - 사용자가 지정한 범위 내에서 집계를 수행
    - 다중 버킷 집계
    - type: "range"
- 날짜 범위 집계
    - 범위 집계: 숫자 값을 범위로 사용
    - 엘라스틱서치에서 지원하는 날짜 형식을 범위로 사용
    - type: "date_range"
- 히스토그램 집계
    - 지정한 범위 내에서 집계를 수행하는 범위 집계와 달리 지정한 수치가 간격을 나타내고 이 간격 범위 내에서 집계를 수행한다.
    - type: "histogram"
- 날짜 히스토그램 집계
    - 분, 시간, 월, 연도를 구간으로 집계 수행
    - tyep: "date_histogram"
- 텀즈(Terms) 집계
    - 버킷이 동적으로 생성되는 다중 버킷 집계
    - 집계시 지정한 필드에 대해 빈도수가 높은 텀의 순위로 결과가 반환
    - 집계할 때에는 반드시 Keyword 데이터 타입의 필드를 사용해야한다.
    - type: "terms"

### 파이프라인 집계
- 다른 집계로 생성된 버킷을 참조해서 집계를 수행
- buckets_path 파라미터를 사용해 참조할 집계의 경로를 지정
- 부모(Parent), 형제(Sibling) 두 가지 유형이 있다.

```
    AGG_SEPARATOR = '>' ;
    METRIC_SEPARATOR = '.' ;
    AGG_NAME = <집계 이름> ;
    METRIC = <메트릭 집계 이름(다중 메트릭 집계인 경우)> ;
    PATH = <AGG_NAME> [ <AGG_SEPARATOR>, <AGG_NAME>]* [ <METRIC_SERARATOR>, <METRIC>] ;
```
1. 형제 집계
    - 기존 버킷에 추가되는 형태가 아니라 새 집계가 생성되는 파이프라인 집계
    - 평균 버킷 집계
        - type: "avg_bucket"
    - 최대 버킷 집계
        - type: "max_bucket"
    - 최소 버킷 집계
        - type: "min_bucket"
    - 합계 버킷 집계
        - type: "sum_bucket"
    - 통계 버킷 집계
        - type: "stats_bucket"
    - 확장 통계 버킷 집계
        - type: "extended_stats_bucket"
    - 백분위수 버킷 집계
        - type: "percentiles_bucket"
    - 이동 평균 버킷 집계
        - type: "moving_avg_bucket"

```
    {
        "aggs": {
            "histo": {
                "date_histogram": {
                    "field": "timestamp",
                    "interval": "minute"
                }
            },
            "aggs": {
                "byte_sum": {
                    "sum": {
                        "field": "bytes"
                    }
                }
            }
        },
        "max_bytes": {
            "max_bucket": {
                "buckets_path": "histo>byte_sum"
            }
        }
    }
```

2. 부모 집계
    - 집계를 통해 생성된 버킷을 사용해 계산을 수행하고, 그 결과를 기존 집계에 반영
    - 선행되는 데이터가 존재하지 않으면 집계를 수행할 수 없음
        - 데이터가 존재하지 않는 부분, 갭(gap) -> 갭 정책 필요
    - 파생 집계
        - type: "derivative"
    - 누적 집계
        - type: "cumulative_sum"
    - 버킷 스크립트 집계
        - type: "bucket_script"
    - 버킷 셀렉터 집계
        - type: "bucket_selector"
    - 시계열 차분 집계
        - type: "serial_diff"

### 근삿값으로 제공되는 집계 연산
- 여러 집계 연산중 실제로 수학적인 계산을 수행하는 것은 매트릭 집계
- 대부분의 집계 연산은 100% 정확한 결과를 제공
- 집계 연산 중 근사값을 제공하는 것은 다음 세가지
    - 카디널리티 집계
    - 백분위 수 집계
    - 백분위 수 랭크 집계
- 분산 환경에서 집계 연산의 여려움
    - 루씬은 분산처리를 지원하지 않음
    - 수치적 계산 결과만을 이용하는 경우에는 값만 전달
    - 최종 결과 계산을 위해 데이터 리스트를 전송해야하는 경우 -> 근사값을 사용


# ElasticSearch Cookbook 3rd 
- [Sample.KR](http://www.acornpub.co.kr/book/elasticsearch-cookbook-3)
- [Sample.US](https://github.com/PacktPublishing/Elasticsearch-5x-Cookbook-Third-Edition)

## 08 집계
- 집계 실행
- stats 집계 실행
- terms 집계 실행
- significant_terms 집계 실행
- range 집계 실행
- histogram 집계 실행
- date_histogram 집계 실행
- filter 집계 실행
- filters 집계 실행
- global 집계 실행
- geo_distance 집계 실행
- children 집계 실행
- nested 집계 실행
- top_hits 집계 실행
- matrix_stats 집계 실행
- geo_bounds 집계 실행
- geo_centroid 집계 실행
