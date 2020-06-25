# Kafka

## kafka topic
- 파티션이 여러개
- 컨슈머가 파티션에서 데이터(record) 가져가도 삭제되지 않음
    - 다른 컨슈머 그룹에서 다시 가져가서 사용가능
    - auto.offset.reset = earliest
- 파티션이 여러개일때 데이터는 어디에 저장?
    - 키가 null이고, 기본 파티셔너 사용할 경우 라운드 로빈
    - 키가 있고, 기본 파티셔너 사용할 경우 키의 해시 값을 구하고, 파티션에 할당
- 파티션의 레코드는 언제 삭제?
    - log.retention.ms : 최대 record 보존 시간
    - log.retention.byte : 최대 record 보존 크기

## Kafka producer
- 토픽에 전송할 메시지를 생성
- 특정 토픽으로 데이터를 publish, 전송할 수 있음
- kafka broker로 데이터를 전송할 때 전송 성공여부를 알 수 있고 실패할 경우, 재시도 할 수도 있음
- serializer (직렬화)
    - key or value를 직렬화하기 위해 사용
        - key : 메시지를 보내면, 토픽의 파티션이 지정될 때 쓰임
        - 카프카는 key를 특정한 hash값으로 변경시켜 파티션과 1대1 매칭을 시킴
        - 토픽에 새로운 파티션을 추가하는 순간 key <-> partition의 일관성은 보장되지 않음
            - 그래서 처음에 잘 생각해서 파티션 개수를 생성하고 추후에 생성하지 않는 것을 추천
    - byte array, string, integer 시리얼라이즈를 사용할 수 있음

## broker, replication, ISR
### broker
- 카프카가 설치되어 있는 서버 단위를 말함
- 보통 3개 이상의 브로커로 구성

### replication
- 복제는 카프카 아키텍처의 핵심
    - partition의 고가용성을 위해 사용 (고가용성이란?)
    - 클러스터에서 서버가 장애가 생길 때 카프카의 가용성을 보장하는 가장 좋은 방법이 복제이기 때문
- partition : 1, replication : 2
    - partition은 원본 1개와 복제본 1개로 총 2개
    - 원본 파티션 : Leader partition
        - 프로듀서가 토픽의 파티션에 데이터를 전달할때 전달받는 주체
        
    - 복제본 파티션 : Follower partition
- 브로커 개수에 따라서 replication 개수가 제한
    - 브로커 개수가 3이면 replication은 4개 이상이 될 수 없음
- 프로듀서에는 ack이라는 상세 옵션이 있음 -> ack를 통해 고가용성을 유지할 수 있음 (replication과 관련이 있는 옵션)
    - ack는 0, 1, all 옵션 3개중 1개를 골라서 설정할 수 있음
        - 0 : Leader partition에 데이터를 전송하고 응답값을 받지 않음, 속도는 빠르지만 데이터 유실 가능성이 있음
        - 1 : Leader partition에 데이터를 전송하고 데이터를 정상적으로 받았는지 응답값을 받음, 나머지 파티션에 복제되었는지는 알 수 없음, 0과 마찬가지로 데이터 유실 가능성이 있음
        - all : 1 옵션에 추가로 follower partition에 복제가 잘 이루어졌는지 응답값을 받음, 데이터 유실가능성은 없지만 속도가 현저히 낮음
- replication 개수가 많아지면? 
    - 브로커의 리소스 사용량도 늘어남
    - 3개 이상의 브로커를 사용할 때 replication은 3으로 설정하는 것을 추천

### ISR
- Leader, Follower partition을 합쳐서 In Sync Replica(ISR) 라고 볼수 있음

##  kafka consumer
- 보통 다른 시스템에서 컨슈머가 데이터를 가저가면 큐 내부 데이터가 사라짐
- 카프카에서는 컨슈머가 데이터를 가져가더라도 데이터가 사라지지 않음
- 토픽의 파티션에 저장된 데이터를 가져오는 것을 polling 폴링이라고 함
- 컨슈머의 역할
    - 토픽의 파티션으로부터 데이터 polling
    - 파티션 오프셋 위치 기록(commit)
    - 컨슈머 그룹을 통해 병렬처리

- 컨슈머 동작 과정
    - 브로커, 컨슈머 그룹, 데이터 직렬화 설정
    - 설정으로 카프카 컨슈머 인스턴스 생성
    - 어느 토픽에서 폴링할건지 설정
        - 특정 파티션에서 가져오도록 설정 가능
    - 폴링 루프
        - poll()메서드를 통해 데이터를 가저옴. 메서드에서 설정한 시간동안 데이터를 기다림
        - 폴링한 records 처리

- 파티션에 들어간 데이터는 파티션 내에서 고유한 번호를 가지게 됨 -> offset
    - offset은 토픽별로, 파티션별로 별개로 지정됨
    - 컨슈머가 데이터르 어느 지점까지 읽었는지 확인하는 용도로 사용
    - 컨슈머가 데이터를 읽기 시작하면 offset을 commit하게 되는데 해당 내용에 대한 정보는 카프카의 __consumer_offset 토픽에 저장됨
        - 컨슈머가 이슈가 생겨서 다시 실행될때 __consumer_offset을 통해 중지되었던 위치부터 이어서 읽을 수 있음

- 컨슈머는 같은 컨슈머 그룹내에서 몇개까지 가능?
    - 여러 파티션을 가진 토픽에 대해서 컨슈머를 병렬처리하고 싶다면 반드시 컨슈머를 파티션 개수보다 적은 개수로 실행시켜야 함
    - 파티션 2개, 컨슈머 1개 : 컨슈머가 2개의 파티션에서 데이터르 가져감
    - 파티션 2개, 컨슈머 2개 : 각 컨슈머가 각각의 파티션을 할당하여 데이터를 가져감
    - 파티션 2개, 컨슈머 3개 : 세번째 컨슈머는 더이상 할당될 파티션이 없어서 동작하지 않음

- 컨슈머 그룹이 다른 컨슈머들의 동작
    - 다른 그룹에 속한 컨슈머들은 다른 컨슈머 그룹에 영항을 주지 않음
    - __consumer_offset 토픽에는 컨슈머 그룹별로, 토픽별로 offset을 나누어 저장하기 때문

## 카프카의 3가지 특징
1. High throughput message capacity
    - 짧은 시간 내에 엄청난 양의 데이터를 컨슈머까지 전달할 수 있음 (파티션, 컨슈머를 통해 병렬처리가 가능하기 때문)
1. Scalability & Fault tolerant
    - 확장성이 뛰어남
    - 이미 사용되고 있는 카프카 브로커가 있다고 하더라도 신규 브로커 서버를 추가해서 수평 확장이 가능함
    - replica로 복제된 데이터로 복구하여 처리할 수 있음
1. Undeleted log
    - 컨슈머가 데이터를 가지고 가더라도 데이터가 사라지지 않음
    - 컨슈머의 그룹 아이디만 다르다면 동일한 데이터도 각각 다른 형태로 처리할 수 있음

## zokeeper 란?
- coordination service (코디네이션 서비스 시스템)
    - 분산된 시스템간의 정보를 공유
    - 컬러스터에 있는 서버들의 상태를 체크
    - 분산된 서버들간에 동기화르 위한 락을 처리