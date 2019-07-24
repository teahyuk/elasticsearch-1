# 7. 검색 확장 기능
## 7.1 Suggest API
### 7.1.1 Term Suggest API
- 비슷한 단어가 있을 경우 제안을 하는 API
- 키워드가 오타가 있을 경우 단어 제안을 하는 기능에 사용될 수 있음
- 편집거리 계산 알고리즘을 이용하여 비슷한 단어들을 제안

### 7.1.2 Completion Suggest API
- 자동완성 기능에 사용될 수 있는 API
- completion 이라는 별도의 데이터 타입을 지정해야 함
- prefix가 일치하는 단어들만 결과로 출력됨

# ETC
## similarity algorithm
- default는 bm25
- 5.0 버전에서 TF/IDF 에서 bm25로 변경

    <figure>
    <iframe src="//www.slideshare.net/slideshow/embed_code/key/VGU2wk2gXqzRC?startSlide=32" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> 
    </figure> 

- https://www.slideshare.net/kjmorc/ss-80803233 (32페이지부터)
- 찾는 검색어가 문서에 많을수록 score가 높으며 전체 문서에 많이 출연한 단어일수록 score 가 낮음

## How Shards Affect Relevance Scoring in Elasticsearch
- https://www.elastic.co/blog/practical-bm25-part-1-how-shards-affect-relevance-scoring-in-elasticsearch

    By default, Elasticsearch calculates scores on a per-shard basis.

- 스코어 계산은 default 로 샤드 기준으로 계산을 함
- 샤드에 document가 어떤 분포로 들어가느냐에 따라 score가 다르게 나올 수 있음

## Function score query
- https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html
- 검색 결과와 임의로 조정한 score 값을 합쳐서 결과를 출력
- 옵션은 score_mode, boost_mode, weight
    - score_mode
        - sub query로 나온 값과 main query로 나온 값을 어떻게 조합할 것인가를 결정
        - multiply(default), sum, avg, first, max, min
- 함수는 field_value_factor, Decay functions, script_score
    

## search_as_you_type
    For more flexible search-as-you-type searches that do not use suggesters, see the search_as_you_type field type.
- 7.2 버전에서 새로 추가됨
- Shingle Token Filter 를 이용하여 기존 필드 이외에 3개의 필드를 더 추가함
    - my_field._2gram
    - my_field._3gram
    - my_field._index_prefix
    #### Shingle Token Filter
        the sentence "please divide this sentence into shingles" might be tokenized into shingles 
        "please divide", "divide this", "this sentence", "sentence into", and "into shingles".  
- 추가된 필드를 이용하여 prefix로 검색 할 수 있고, 중간에서 있는 값도 검색이 가능함
- 공식 문서의 예제에서는 'multi_match' 와 'match_bool_prefix'를 조합하여 사용
    ### match_bool_prefix
        if message is 'quick brown f', then query is
        GET /_search
        {
            "query": {
                "bool" : {
                    "should": [
                        { "term": { "message": "quick" }},
                        { "term": { "message": "brown" }},
                        { "prefix": { "message": "f"}}
                    ]
                }
            }
        }
