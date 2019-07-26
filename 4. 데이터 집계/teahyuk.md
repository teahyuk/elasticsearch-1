# 데이터 집계

## 어그리계이션

어그리게이션 사용법

```json
{
  "aggs" : {
      "<aggregation_name>" : {
          "<aggregation_type>" : {
              <aggregation_body>
          }
          [,"meta" : {  [<meta_data_body>] } ]?
          [,"aggregations" : { [<sub_aggregation>]+ } ]?
      }
      [,"<aggregation_name_2>" : { ... } ]*
  }
}
```

트리구조로 치면 아래와 같다

- aggs
  - aggs_name *
    - aggs_type
      - aggs_body
    - meta ?
    - aggs ?
      - sub aggs +

여러 aggs 를 만들 수 있고, 해당 aggs는 한개의 aggs타입만 가질 수 있으며 sub-aggs를 가질수 있는 aggregation들이 있기때문에 하위 aggs를 계속 생성 가능하다.

출력은 설정한 aggs이름을 중심으로 튀어나온다.

출력

```json
{
    ...
    "aggregations": [
        "<aggregation_name>": {
            <aggregation_body>
        },
        "<aggregation_name_2>": {
            <aggregation_body_2>
        }
    ]
}
```

---
어그리게이션은 집계함수로써 다음과 같은 종류가 있다

- Bucketing
- Metric
- Metrix
- Pipeline

### [BucketAggregation](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-bucket.html)

- 어그리게이션에 주어진 조건에 해당하는 doc를 버킷이라는 단위로 구분해서 담아 새로운 집합을 형성하는 방법이다.
- RDBMS에서 보면 inner Query쯤 되시겠다. `view`정도라고하면 나으려나?
- 참고로 Select절에 들어가는 집계함수의 기능과는 다르다.

### [Metrics Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics.html)

- 계산 aggregation이라고 보면됨.
- RDBMS에서는 Select구문에 많이 사용하는 집계함수와 가장 비슷함.
- min,max,sum,avg,percentiles 등등이 있음.

### [Matrix Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-matrix.html)

- 실험적 기능이다. (제거가능성 존재)
- 여러 Metrics Aggregation들을 묶어서 행렬로 출력하는 Aggs다.
- 현재 종류는 1가지 이다.

#### Matrix Stats Aggregation

- 각 필드들을 집계한 상태 정보를 행렬로 뿌려주는 aggregation

요청

``` json
{
    "aggs": {
        "statistics": {
            "matrix_stats": {
                "fields": ["poverty", "income"]
            }
        }
    }
}
```

응답

``` json
{
    ...
    "aggregations": {
        "statistics": {
            "doc_count": 50,
            "fields": [{
                "name": "income",
                "count": 50,
                "mean": 51985.1,
                "variance": 7.383377037755103E7,
                "skewness": 0.5595114003506483,
                "kurtosis": 2.5692365287787124,
                "covariance": {
                    "income": 7.383377037755103E7,
                    "poverty": -21093.65836734694
                },
                "correlation": {
                    "income": 1.0,
                    "poverty": -0.8352655256272504
                }
            }, {
                "name": "poverty",
                "count": 50,
                "mean": 12.732000000000001,
                "variance": 8.637730612244896,
                "skewness": 0.4516049811903419,
                "kurtosis": 2.8615929677997767,
                "covariance": {
                    "income": -21093.65836734694,
                    "poverty": 8.637730612244896
                },
                "correlation": {
                    "income": -0.8352655256272504,
                    "poverty": 1.0
                }
            }]
        }
    }
}
```

### [Pipeline Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-pipeline.html)

상위에서 만든 aggregation의 출력들을 모아서 그것들을 가지고 재 집계하는 기능들이다.

Bucket Aggregation들 가지고 하위 aggs를 만들어서 몇단계로 만들다보면 대체 할 수 있지만 쿼리과정도 복잡해지고 성능도 안좋아져서 제약을 좀 더 늘리고 이런 집계기능들을 만든 것 같다.

`bucket_path`라는 특별한 구문을 사용하는 옵션 필드를 사용해 이전에 만들어진 aggregation들을 이용한다.

`bucket_path` 구문

```html
AGG_SEPARATOR       =  '>' ;
METRIC_SEPARATOR    =  '.' ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
PATH                =  <AGG_NAME> [ <AGG_SEPARATOR>, <AGG_NAME> ]* [ <METRIC_SEPARATOR>, <METRIC> ] ;
```

예제

```json
{
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
                }
            }
        },
        "max_monthly_sales": {
            "max_bucket": {
                "buckets_path": "sales_per_month>sales" 
            }
        }
    }
}
```

- `sale_per_month`이라는 `date_histogram`집계를 먼저 사용하고 내부에 `sales`라는 `sum`집계를 나타내는데 `max_bucket`이라는 pipeline aggs를 사용해서 `sales` 집계결과의 최대값을 따로 나타낸다.

#### spacial paths

- `_`로 시작하는 내부 필드나 특별한 몇몇 기능도 사용 가능하다
  - ex) `_bucket_count`

예제

```json
{
    "aggs": {
        "my_date_histo": {
            "date_histogram": {
                "field":"timestamp",
                "calendar_interval":"day"
            },
            "aggs": {
                "the_movavg": {
                    "moving_avg": { "buckets_path": "_count" } 
                }
            }
        }
    }
}
```

- `_count`값을 source로 하여 `moving_avg` 집계결과를 나타낸다.

---

## 그 외 기능

### 캐싱

- 자주 사용 하는 aggregation 캐시 가능
  - 단 캐싱하기위해서는 출력 시 doc들을 출력하게 하면 안된다.

> aggregation 출력 시 doc들을 뽑지 않는 옵션 : `size=0`

### 메타데이터

- 각 aggregation 마다 메타 데이터 설정 가능
  - 태그처럼 붙어 있으며 말그대로 메타데이터임

### 출력 시 aggregation 타입 출력

- urlParam 으로 `typed_keys` 추가.

> 출력 모양 `<type>#<aggregation_name>`
