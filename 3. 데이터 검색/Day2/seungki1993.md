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

### Standard Analuzer
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

### whiespace 분석기
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
