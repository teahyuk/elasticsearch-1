#### 엘라스틱 서치 고려 설정 사항
-   하드웨어 선택
    -   엘라스틱 서치는 역색인을 통해 저장된 data를 가지고 메모리에서 많은 작업을 수행한다.
    -   즉 메모리가 크면 클수록 성능향상에 도움을 주나 항상 좋은건 아니다. 
    -   그 이유는 엘라스틱 서치는 결국 JVM위에서 돌아가는데 HEAP 메모리를 32GB이상 지정할 수 없다.
    -   결론은 무작정 좋은 하드웨어를 사용하기 보다는 테스트를 통해 적절한 크기를 찾고 이후 추가하는 방식으로 운영해야 한다.
    
-   JVM HEAP 크기
    -   일단 HEAP 메모리가 크다는 것은 대량의 데이터를 메모리에 저장하고 빠르게 접근할 수 있다.
    -   그럼 크면 클수록 좋은가?
        -   위에서 말했듯이 엘라스틱처치는 jvm위에서 작동하고 결국 GC의 과정을 겪게 된다.
        -   GC가 시작되면 모든 노드의 처리 작업은 중지가 되며 GC가 끝나기만을 기다린다.
        -   즉 HEAP이 너무 크면 그만큼 정지 시간도 늘어난다.
    -   올바른 JVM HEAP 크기는?
        -   시스템 전체의 메모리의 절반만 사용하도록 한다.
            -   루씬은 기본적으로 메모리내에 데이터를 캐싱하도록 설계되었다. 그렇기에 엘라스틱서치에 너무 많은 메모리를 주게
                되면 루씬이 사용할 메모리가 없기에 성능에 심각한 영향을 줄 수 있다.
        -   compressed oop를 사용할 수 있도록 32GB 이하로 사용한다.
    - compressed oop란?
        -   heap에 올라가있는 데이터를 object라고 부르고 objct의 메모리상의 주소를 ordinary object pointer라는 구조체에 저장한다.
        -   oop는 32bit일때는 최대 4GB , 64bit 18EB까지 주소 공간을 가리 킬 수 있다. 그러나 64bit 경우 포이터 자체의 크기가 32bit보다 크기에 성능이 떨어진다.
        -   그래서 jvm은 64bit 시스템이여도 heap 영역이 4GB다 작으면 32bit 기반의 oop를 사용한다
            -   그러나 32bit기반의 oop로는 4GB를 넘어가는 object는 표현 할 수 없기에 jvm은 Compressed oop를 사용한다.
        -   compressed oop는 객체의 주소가 아닌 오프세을 가르키게 하여 기존보다 8배의 더 많은 주소 공간을 표시 할 수 있다.
            -   객체의 패딩을 추가하여 8바이트의 배수가 되도록한다.
            -   즉 패딩을 사용하게 되면 oops의 마자믹 세 비트는 항상 0이다.(8의 배수는 이진수로 000으로 끝나기 때문에)     

-   스와핑 비 활성화
    -   대부분의 운영체제는 메모리를 효과적으로 사용하고 위해 메모리 스와핑을 한다.
    -   스와핑은 노드의 안정성에 큰 영향을 주며 성능에도 좋지 못하다.
    -   그렇게이 스와핑을 비활성화 해야한다.
        -   sudo swapoff -a
        -   elasticsearch.yml --> bootstrap.memory_lock: true
 
-   쓰레드 풀 및 가비지 컬렉터
    -   엘라스틱 서치는 색인 , 검색 , 정렬 , 집계등 다양한 연산을 수행하기 위헤 Thread pool을 사용한다.
        -   스레드 풀 설정을 변경하지 않는 것을 권장한다고 한다.
    -    연산에 따라 사용하는 쓰레드풀이 다르다
    - Thread pool type
        -   fixed 고정된 크기의 쓰레드 풀
        -   fixed_auto_queue_size
            -   쓰레드는 고정된 크기를 가지고 있으나 옵션에 따라 가변적인 queue 사이즈를 가지고 있다.
         -  scaling 
            -   쓰레드 개수가 동적인 thread pool type
     -  gc 또한 변경하지 않는 것을 권장한다.
### 엘라스틱서치 관리 및 모니터링
-   클러스터 활동 관리 및 모니터링은 사용자 스스로 해야한다.
#### cluster health
-   클러스터의 상태 정보를 확인 할 수 있다
```
curl -X GET "localhost:9200/_cluster/health"
```
```
 "cluster_name" : "my-application",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 14,
  "active_shards" : 14,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 14,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0

```

-   cluster 상태
   -    red
        -  특정 혹은 일부 인덱스의 pri , replica 샤드가 제대로 동작하고 있지 않다.
    -   yellow
        -   일부 혹은 모든 인덱스의 replica 샤드가 정상적으로 동작하고 있지 않는 상태
    -   green
        -   정상적으로 작동하고 있다.
    - green , yellow 상태는 성능에는 영향이 있으나 모든 인덱스 쓰기  / 읽기에는 이상이 없다.

#### cluster state
-   클러스터에 모든 메타 데이터 정보를 확인 할 수 있다.
-   클러스터에 있는 인덱스 정보 , 샤드의 위치 , 노드 정보 등
-   해당 정보는 빠르게 접근하기 위해 메모리에 보관된다. 즉 대량의 인덱스 및 샤드가 많아지면 많아 질수록 메모리 사용량은 커진다.
    -   그 이유는 클러스터 state는 인덱스와 샤드 정보를 가지고 있기 때문이다.
    -   또한 단일 쓰레드로 업데이트작업을 수행한다.
```
curl -X GET "localhost:9200/_cluster/state"
```
#### cluster stats
-   클러스터의 전체 통계를 확인 할 수 있다.
-   OS , JVM 버전 , 메모리 사용량 , 세그먼트수 등 


#### pending cluster task
-   클러스터에서 작업량이 많거나 비정상적인 상황으로 인해 밀리고 있는 작업을 확인 할 수 있는 API다
-   해당 api는 cluster에 status가 green임에도 불구하고 포퍼먼스가 안나올때 가장 먼저 체크 하면 좋다.

#### Node hot thread
-   엘라스틱 서치는 실행 중인 모든 스레드를 얻어 각 스레드가 사용한 리소스 , 
대기 상태 횟수 , 차단된 시간 등 다양한 정보를 수집한다.
    -   해당 작업은 반복하여 수행
    - 가장 오랜 시간 실행 중인 쓰레드가 맨우에 오도록 내림차순 정렬
-   hot thread란?
    -   각 노드에서 가장 많은 리소스를 잡아 먹는 thread
-   해당 api를 통해 각 노드의 hot thread를 알 수있다.
-   즉 갑작스런 성능 저하가 생긴다면 해당 api를 통해서 성능저하가 일어나는 곳을 유추 할 수 있다.

#### cluster reroute
-   샤드 할당을 수동으로 하는 API
-   샤드를 다른 노드로 명시적으로 이동 할 수 있으며 , 할당 되지 않는 샤드를 특정 노드에 할당 할 수 있다.
- rerout를 한 후 엘라스틱 서치가 재조정을 할때 다시 원상 태로 돌아갈 수 있다. 즉 샤드가 이동되었던 곳으로 되돌아 간다.
- dry_run 쿼리 스트링을 붙여서 보낸다면 reroute 가 되었을때 클러스터 상태를 반환 받을 수 있으며
    reroute는 실행이 되지는 않는다.
    
#### cluster update setting
```
PUT /_cluster/settings
{
    "persistent" : { // 영구적으로 변경 restart해도 반영이 되어있다.
        "indices.recovery.max_bytes_per_sec" : "50mb"
    }
}
```

```
PUT /_cluster/settings?flat_settings=true
{
    "transient" : { //restart 하면 최기화
        "indices.recovery.max_bytes_per_sec" : "20mb"
    }
}
```

- 해당 api로 클러스터의 세팅을 변경 할 수 있으며 각각 우선 순위가 존재한다
    1. transient
    2. persistent 
    3. elasticsearch.yml

#### node stats
-   cluster stats랑 유사함
-   node의 사용 메모리 , jvm , os , translog 등 다양한 정보를 확인 할 수 있다.

#### cluster allocation Explain
-   할당 되지 않은 샤드에 경우 할당 해제된 이유 , 할당된 샤드의 경우 다른 노드로 이동 되지 않은 이유에 대해 정보 제공 api
-   include_disk_info 옵션은 디스크 사용량 및 샤드크, 샤드 위치 등을 반환한다.

### 백업 복구의 중요
-   정기적인 백업 절차를 설정하기위해 스냅샷을 저장할 저장소를 선정해야한다.
    -   yml path.repo로 설정
-   정기적인 백업은 필요로 하다

### alias
- 실서버에서 인덱스에 오류가 발생하거나 매핑정보가 변경된다면 문제가 크다  인덱스를 재설정 할 때
까지 지속적인 오류가 발생한다.
- 이를 방지하기 위해 alias가 기능을 제공하며 멀티테넌시도 쉽게 가능하다.
- 즉 인데스가 추가되고 삭제되어도 클라 입장에서는 동일한 인덱스에 요청을 보낸다고 생각할 것이다.
- filter을 통해 원하는 필드 값만을 걸러내서 별칭을 만들 수 도 있다.
- 여러개의 인덱스로 하나의 별칭을 만들 수 있는데 is_write_index true라는 옵션을 인덱스에 주면 해당 인덱스에만 쓰기 작업이 가능
    - 실험결과 하나만 인덱스 중 하나만 가능하다
    - 모두 false이면 write는 안된다.


### 인덱스 템플릿 설정
-   인덱스 생성시 인덱스 탬플릿을 통해 인덱스를 생성 할 수 있다.
-   템플릿은 도중에 변경이 되어도 이미 이를 통해 만들어진 인덱스에는 영향이 없다.
-   템플릿이 여러개가 있다면 order 옵션을 통해 먼저 적용될 템플릿을 설정 할 수 있다.
```
curl -X PUT "localhost:9200/_template/template_1" -H 'Content-Type: application/json' -d'
{
  "index_patterns": ["te*", "bar*"], // 해당 패턴에 해당하는 인덱스는 template를 통해 만들어진다.
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
      "host_name": {
        "type": "keyword"
      },
      "created_at": {
        "type": "date",
        "format": "EEE MMM dd HH:mm:ss Z yyyy"
      }
    }
  }
}
'

```
 
 ###엘라스틱 서치 병렬 치리 단위
 -  인덱스에서 병렬 처리 수준을 결정하는 것은 샤드의 개수
 -  샤드의 개수가 많을 수록 병렬 처리는 늘어나고 각 샤드가 상대적으로 더 적은 작업을 수행 할 수 있다.
 -  그럼 샤드의 개수를 무작정 늘리는것이 좋은가?
    -   샤드의 개수가 많아지면 유사도에 큰 영향을 미친다.
    -   유사도 검색은 모든 샤드를 대상으로 하는 것이아니라 각 샤드의 컨텍스트 안에서 수행되기 때문이다.
    -   집계 정확도에도 옇향을 미친다.
    -   즉 샤드가 꼭 많다고 좋은 것은 아니다.

-   올바른 샤드 구성 방법
    -   샤드의 최대 크기에는 제한이 없지만 50GB를 넘지 않는 것이 좋다
    -   적절한 샤드를 구성하기 위해서는 실제 데이터와 쿼리로 테스트를 해보는 것이 좋다.
-   elastic blog
    -   https://www.elastic.co/kr/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster
    
#### 시간 기반 인덱스
-   고정된 주기로 인덱스를 만들어 저장하는 방식이다.
    -   시간 기반 인덱스는 데이터 보관 주기를 정교하게 관리 할 수 있다.
-   그러나 데이터 인덱싱량이 빈번하게 변하는 경우에는 균일한 목표 샤드 크기를 유지하는것이 어렵다
    -   이러한 것을 통제하기 위해 Rollover 및 shrink api가 존제
-   Rollover 인덱스 api
    -   클러스터가 저장해야하는 도큐먼트와 인덱스의 개수를 지정하거나 , 저장해야 할 도큐먼트의 최대 기간을 지정할 수 있다.
    -   주어진 조건을 넘어서면 신규인덱스를 생성하고 별칭을 신규 인덱스로 전환
-   Shrink 인덱스 api
    -   기존 인덱스를 더 적은 개수의 프라이머리 샤드를 가진 신규 인덱스로 변경
    