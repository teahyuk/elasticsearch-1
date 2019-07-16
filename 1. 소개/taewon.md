## 엘라스틱서치 장점
  - 전문 검색
  - 스키마리스
  - 멀티테넌시 
    * 같은 필드를 검색할 경우 여러개의 인덱스에서 검색 가능
  - 역색인

## 엘라스틱서치 단점
  - Near Realtime
    * 역색인이 완료된 상황에서 검색이 가능하기 때문에 완전한 실시간은 아님
  - 트랜잭션과 롤백 기능을 제공하지 않음
    * Version으로 업데이트 여부를 확인할 수 있으나 트랜잭션을 보장하지는 않음
  - 데이터의 업데이트를 제공하지 않음
    * 데이터를 업데이트 할 경우 Delete 후 새로 Insert하는 과정을 거치기 때문에 리소스가 많이 사용됨

## 설치시 주의사항
  - 파일의 권한 문제 때문에 실행이 되지 않을 경우
    * 리눅스에 'elastic' 이라는 유저를 만들어서 유저를 변환한 후 실행해야 함
  - vm.max_map_count 설정 에러
    * Docker 에서 실행할 경우 생길 수 있는 에러
        > max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    * 호스트의 sysctl.conf 의 설정 값을 변경하면 해결
        >$ grep vm.max_map_count /etc/sysctl.conf \
        vm.max_map_count=262144
  - ElasticSearch의 Heap Size는 32G 이하로 설정
    * 자바의 Ordinary Object Pointer의 성능 이슈로 인하여 32G 이상으로 Heap Size를 설정하면 성능이 떨어진다고 함