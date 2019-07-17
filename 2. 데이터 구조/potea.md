# ElasticSearch Cookbook 3rd 
- [Sample.KR](http://www.acornpub.co.kr/book/elasticsearch-cookbook-3)
- [Sample.US](https://github.com/PacktPublishing/Elasticsearch-5x-Cookbook-Third-Edition)

## 03 매핑 관리

- 명시적 매핑 생성 사용
- 기본 타입 매핑
- 배열 매핑
- 객체 매핑
- 도큐먼트 매핑
- 도큐먼트 매핑에서 동적 템플릿 사용
- 중첩 객체 매핑
- 자식 도큐먼트 관리
- 필드에 다중 매핑 추가


## 04 기본 작업

- 색인 생성, 색인 삭제, 색인 열고 닫기
- 색인에 매핑 입력, 매핑 조회
- 색인 재생성, 색인 새로 고침, 색인 플러시, 색인 강제 병합, 색인 축소
- 색인과 타입 존재 여부 확인
- 색인 설정 관리, 색인 앨리어스 사용, 색인 롤오버
- 도큐먼트 색인, 도큐먼트 조회, 도큐먼트 삭제, 도큐먼트 업데이트
- 원자성 작업 속도 향상 (벌크 작업)
- GET 작업 속도 향상 (다중 GET)

### 색인 생성
- 데이터의 주 컨테이너라고 할 수 있는 색인을 만드는 것
    ```
        http://<server>/<index_name>
        curl -XPUT http://127.0.0.1:9200/myindex -d '{JSON}'
    ```

### 색인 삭제
- 색인의 샤드, 매핑, 데이터를 삭제
    ```
        http://<server>/<index_name>
        curl -XDELETE http://127.0.0.1:9200/myindex
    ```

### 색인 열고 닫기
- 데이터를 유지하면서 자원을 절약하는 방법
    ```
        curl -XPOST http://127.0.0.1:9200/myindex/_close
    ```

### 색인에 매핑 입력
- SQL 테이블 생성과 유사한 개념
    ```
        http://<server>/<index_name>/<type_name>/_mapping
        curl -XPUT 'http://localhost:9200/myindex/order/_mapping' -d '{JSON}'
    ```

### 매핑 조회
- 매핑을 분석할 필요가 있을 때 사용
    ```
        http://<server>/_mapping
        http://<server>/<index_name>/_mapping
        http://<server>/<index_name>/<type_name>/_mapping
        curl -XGET 'http://localhost:9200/myindex/order/_mapping?pretty=true'
    ```
-매핑은 일래스틱서치가 클러스터 수준으로 저장한다.

### 색인 재생성
- 매핑 제약으로 정의된 매핑은 삭제할 수 없기에 다시 색인해야한다.
    ```
        http://<server>/_reindex
        curl -XPOST 'http://localhost:9200/_reindex?pretty=true' -d '{JSON}'
    ```

### 색인 새로 고침
- 색인 강제 새로 고침을 통해 검색자의 상태를 제어할 수 있다.
    ```
        http://<server>/<index_name(s)>/_refresh
        curl -XPOST 'http://localhost:9200/myindex/_refresh'
    ```
- 클러스터의 모든 색인을 새로고침
    ```
        http://<server>/_refresh
    ```
- 준 실시간(NRT, Near Real-Time) 기능은 자동으로 관리되며 데이터를 변경하면 매초 자동으로 색인을 새로 고친다.
- 파일디스크립터와 관련한 과도한 I/O를 방지하기 위해서 도큐먼트를 입력할 때 마다 색인 상태를 새로 고치지는 않는다.

### 색인 플러시
- 가용 메모리 확보와 트랜잭션 로그를 비우기 위해 사용
- 정기적으로 플러시하지만 강제 플러시가 유용한 경우가 있다.
    ```
        http://<server>/<index_name(s)>/_flush[?refresh=True]
        curl -XPOST 'http://localhost:9200/myindex/_flush?refresh=true'
    ```
- 클러스터의 모든 색인을 플러시
    ```
        http:///_flush[?refresh=True]
    ```

### 색인 강제 병합
- 루씬: 디스크에 세그먼트로 데이터를 저장
- 강제 병합 작업은 색인을 통합해 세그먼트 수를 줄이고 검색 성능을 빠르게 한다.
    ```
        http://<server>/<index_name(s)>/_forcemerge
        curl -XPOST 'http://localhost:9200/myindex/_forcemerge'
    ```
- 클러스터의 모든 색인을 최적화
    ```
        http://<server>/_forcemerge
    ```
- 일래스틱서치에서 도큐먼트 삭제는 디스크에서 제거하는 것이 아니고, 삭제표시(tombstone)하는 것으로 가용 공간을 늘리려면 삭제된 도큐먼트를 제거하기 위해 강제 병합해야한다.

### 색인 축소
- 색인의 샤드 개수를 줄이는 축소 API를 통해 최적화하는 방법 제공
    ```
        http://<server>/<source_index_name>/_shrink/<target_index_name>

    ```
- (명령어는 간단하지만...사용단계는 책 참조 p.167)

### 색인과 타입 존재 여부 확인
- 존재하지 않는 색인이나 타입에 쿼리하는 에러가 발생할 수 있다.
    ```
        http://<server>/<index_name>/
        curl -i -XHEAD 'http://localhost:9200/myindex/' 
        http://<server>/<index_name>/<type>/
        curl -i -XHEAD 'http://localhost:9200/myindex/order/'
    ```

### 색인 설정 관리
- 샤딩, 레플리카, 캐시, 텀(term) 관리, 라우팅, 분석 등의 기능을 제어
    ```
        http://<server>/<index_name>/_settings
        curl -XGET 'http://localhost:9200/myindex/_settings?pretty=true'
        curl -XPUT 'http://localhost:9200/myindex/_settings' -d '{JSON}'
    ```

### 색인 앨리어스 사용
- 앨리어스를 이용하여 공통 이름으로 그룹을 만들 수 있다.
    ```
        http://<server>/_aliases
        http://<server>/<index>/_alias/<alias_name>
    ```
- 주로 다중 색인에 데이터를 저장할 때 색인을 관리하는 단순한 기능적인 구조.

### 색인 롤오버
- 로그를 관리하는 시스템을 사용할 때 유용
- 검사할 조건을 정의하면 자동으로 새 색인을 롤링하고 앨리어스를 통해 가상 색인만 참조할 수 있다.

### 도큐먼트 색인
- 루씬과 일래스틱서치에서 갱신은 교체를 의미한다.

    | Method | URL |
    | - | - |
    | POST | ```http://<server>/<index_name>/<type>``` |
    | PUT/POST | ```http://<server>/<index_name>/<type>/<id>``` |
    | PUT/POST | ```http://<server>/<index_name>/<type>/<id>/_create``` |

### 도큐먼트 조회(GET)/삭제(DELETE)
- GET REST 호출을 통해 실시간으로 도큐먼트를 가져올 수 있다.
- HTTP 메소드만 DELETE로 하면 삭제 가능
    ```
        http://<server>/<index_name>/<type_name>/<id>
        curl -XGET 'http://localhost:9200/myindex/order/2qLrAfPVQvCRMe7Ku8r0Tw?pretty=true'
        curl -XDELETE 'http://localhost:9200/myindex/order/2qLrAfPVQvCRMe7Ku8r0Tw'
    ```

### 도큐먼트 업데이트
- 업데이트를 하는 두가지 방법, 새 도큐먼트 추가, 업데이트 호출
- 색인에 비해 업데이트의 장점은 네트워크 사용 감소
    ```
        http://<server>/<index_name>/<type_name>/<id>/_update
        curl -XPOST 'http://localhost:9200/myindex/order/2qLrAfPVQvCRMe7Ku8r0Tw/_update?pretty' -d '{JSON}'
    ```

### 원자성 작업 속도 향상 (벌크 작업)
- HTTP의 오버헤드로 속도 향상을 위해 벌크 CRUD 호출을 실행 할 수 있다.
    ```
        http://<server>/<index_name>/_bulk
        curl -s -XPOST localhost:9200/_bulk --data-binary @bulkdata;
    ```

### GET 작업 속도 향상 (다중 GET)
- 여러 개의 ID로 많은 도큐먼트를 가져오기 위해 다중 GET을 지원한다.
    ```
        http://<server>/_mget
        http://<server>/<index_name>/_mget
        http://<server>/<index_name>/<type_name>/_mget
        curl -XPOST 'localhost:9200/_mget' -d '{JSON}'
    ```
- JSON에 색인, 타입, ID 등을 제공해야한다.
