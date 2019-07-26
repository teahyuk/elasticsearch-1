# 5. 집계
## 종류
- [Metrics Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics.html)
- [Bucket Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket.html)
- [Pipeline Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-pipeline.html)
- [Matrix Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-matrix.html)

## 1. Metrics Aggregations
- 숫자 연산을 할 수 있는 값들에 대한 집계를 수행
- a single numeric metric OR multiple metrics
- script 지원
### 1-1. Cardinality Aggregation
- 중복된 값은 제외한 고유한 값에 대한 집계를 수행
- HyperLogLog++ 알고리즘을 기반으로 동작
- 값이 정확하지 않을 수 있음
- precision_threshold 속성을 설정하여 정확도를 조정할 수 있음
```
- 정학성을 위해 메모리를 교환하는 방법을 결정, 이는 정확도를 높일수록 더 많은 메모리를 필요로 함
- 카디널리티가 낮은 집합일수록 더 뛰어난 정확성을 보임
- 수십억개의 고유값이 존재하더라도 메모리 사용은 정확도 설정에 의존해서 고정된 메모리를 사용
```
- threshold 값이 100 정도에서 에러의 비율은 1~6% 정도
![Alt text](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/cardinality_error.png)

### 1-2. Percentiles Aggregation
- 특정 값이 어디에 얼마나 분포되어있는지를 보여주는 집계
- TDiest 알고리즘을 사용
- 항상 정확한 값이 나오는 것은 아니며 공식 홈페이지에 나오는 주의 사항을 참조하는 것이 좋음
```
- Accuracy is proportional to q(1-q). This means that extreme percentiles (e.g. 99%) are more accurate than less extreme percentiles, such as the median
  -> 정확도는 q (1-q)에 비례합니다. 이는 극한 백분위 수 (예 : 99 %)가 중간 값 백분위 수보다 정확하다는 것을 의미합니다
- For small sets of values, percentiles are highly accurate (and potentially 100% accurate if the data is small enough).
  -> 작은 값 집합의 경우 백분위 수는 매우 정확합니다 (데이터가 충분히 작 으면 100 % 정확할 수 있습니다).
- As the quantity of values in a bucket grows, the algorithm begins to approximate the percentiles. It is effectively trading accuracy for memory savings. The exact level of inaccuracy is difficult to generalize, since it depends on your data distribution and volume of data being aggregated
  -> 버킷의 값의 양이 증가함에 따라 알고리즘은 백분위를 근사하기 시작합니다. 이것은 효과적으로 메모리 절감을위한 trade off 입니다. 정확도 수준은 데이터 분포 및 집계되는 데이터의 양에 따라 달라지므로 정확히 일반화하기는 어렵습니다.
```
 ![Alt text](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/percentiles_error.png)
 - compression 옵션을 이용하여 정확도를 높일 수 있으며 정확도가 올라감에 따라 메모리를 더 사용하게 됨
 
 ## 2. Bucket Aggregations
 - 버킷을 생성한 후 버킷에 따라 결과를 측정함
 - 데이터 집합을 메모리에 올리기 때문에 단계가 깊어질수록 메모리 사용량이 올라감
 ### 2-1. Terms Aggregations
 - keyword 데이터 타입에서 대당 term 이 몇번 나오는지에 대한 통계
 - 각 샤드에서 한번 Agg 된 결과를 정리해서 결과를 반환하기 때문에 count에 대한 값이 정확하지 않을 수 있음
``` 
 각 샤드에 값이 아래와 같이 존재할 경우

| rank | Shard A | Shard B | Shard C |
| :---: | :---: | :---: | :---: |
| 1 | Product A (25) | Product A (30) | Product A (45) |
| 2 | Product B (18) | Product B (25) | Product C (44) |
| 3 | Product C (6) | Product F (17) | Product Z (36) |
| 4 | Product D (3) | Product Z (16) | Product G (30) |
| 5 | Product E (2) | Product G (15) | Product E (29) |
| 6 | Product F (2) | Product H (14) | Product H (28) |
| 7 | Product G (2) | Product I (10) | Product Q (2) |
| 8 | Product H (2) | Product Q (6) | Product D (1) |
| 9 | Product I (1) | Product J (8) |   |
| 10 | Product J (1) | Product C (4) |   |

상위 5개의 값에 대해서는 결과가 아래와 같이 나온다

| rank | Shard A | Shard B | Shard C |
| :---: | :---: | :---: | :---: |
| 1 | Product A (25) | Product A (30) | Product A (45) |
| 2 | Product B (18) | Product B (25) | Product C (44) |
| 3 | Product C (6) | Product F (17) | Product Z (36) |
| 4 | Product D (3) | Product Z (16) | Product G (30) |
| 5 | Product E (2) | Product G (15) | Product E (29) |

각 샤드에서 반환된 값을 합쳐서 결과를 만들기 때문에 최종 결과는 다음과 같이 나온다

| rank | Product |
| :---: | :---: |
| 1 | Product A (100) |
| 2 | Product Z (52) | 
| 3 | Product C (50) | 
| 4 | Product G (45) | 
| 5 | Product B (43) |

Product C 의 경우에는 원래 값이 54가 나와야 하는데 b shard의 결과에서 상위 5개에 들지 못했기 때문에 값이 누락되어서 들어간다
따라서 정확한 값을 얻고 싶을 경우, 각 샤드에서 반환되는 결과의 size 값을 늘려야 보다 정확한 값을 얻을 수 있다   
```

- 각 샤드별로 반환되는 값의 default는 (size * 1.5 + 10)
