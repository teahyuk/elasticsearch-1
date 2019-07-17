## mapping
  - 생성된 후에는 변경이 불가능함
  - 추가는 가능함
  - 잘못 매핑되어 있을 경우 새로 매핑을 추해서 땜빵은 가능함
  - 매핑 수정하려면 reindex 해야함
  - index를 timebase로 생성하는 가장 큰 이유라고 생각
  - mapping alias를 이용하여 검색되는 데이터를 동적으로 수정 가능함

## 데이터 타입
### keyword
  - String 데이터 넣으면 자동으로 생김
  - tokenize 하지 않은 원본 데이터
  - agg 는 keyword만 가능하니 통계 작업해야 하는 데이터는 필수
  - mapping을 customize 해서 넣을 경우 multi field로 keyword를 정의하지 않을 경우 안생김
### Array
  - 기본 데이터 타입은 아님
  - 데이터가 있고 거기다가 데이터를 추가 할 경우 자동으로 array로 변경됨
  - 하나의 key 값에 여러개의 데이터가 들어가서 검색이 이상하게 됨
### nested
  - array 타입이 검색이 이상하게 되는 부분을 보안하기 위해 만든 타입
  - Object가 정확히 일치해야만 검색에 걸림

### Analyzer
  - 데이터를 쪼개는 방식을 정의
  - Character filter, tokenizer filter, token filter로 구성
  - analyzer에서 tokenizer만 있으면 동작하고 나머지는 option
    #### character filter
    - input data를 제일먼저 바꿔주는 filter
    - 몇개 없음
    ### tokenizer filter
    - character filter를 통해 변환된 데이터를 쪼갬
    - 많아서 쓸때마다 챙겨봐야 하는 아이
    ### token filter
    - tokenizer filter를 통해 쪼개는 데이터를 변환하는 filter

### 동의어 사전
  - 동의어 사전을 만들어서 추가 가능함
  - 동적을 변경 가능