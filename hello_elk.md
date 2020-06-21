# ELK 사전조사
- - -

+ https://victorydntmd.tistory.com/308
+ https://iassad.tistory.com/7
+ https://heowc.tistory.com/49

### Elastic Stack
- 사용자가 서버나 데이터베이스로부터 원하는 데이터를 실시간으로 수집하고 검색, 분석하여 시각화 시키는 오픈소스 서비스
- ELK = Elasticsearch + Logstash + Kibana (+ Beats)

## Elasticsearch
Lucene 검색엔진 기반의 데이터베이스로 고성능의 검색 기능, 대규모 분산 시스템 기능 등을 제공한다. 표준 RESTful API와 JSON을 이용해 데이터를 처리한다. <br>
- **Scale out** : 샤드를 통해 규모가 수평적으로 늘어날 수 있음
- **고가용성** : Replica를 통해 데이터의 안정성을 보장
- **Schema Free** : Json 문서를 통해 데이터 검색을 수행하므로 스키마 개념이 없음

### 비교
| 관계형 | ES |
|:--------|:--------|
| Database | Index |
| Table | Type |
| Row | Document |
| Column | Field |
| Index | Analyze |
| Schema | Mapping |
| Physical partition | Shard |
| Primary key | _id |
|Relational | Parent/Child |

### 구조
1. **Cluster**
    - Elasticsearch에서 가장 큰 시스템 단위를 의미
    - 최소 하나 이상의 노드로 이루어진 노드들의 집합
    - 여러 대의 서버가 하나의 클러스터를 구성할 수 있음
    - 한 서버에 여러 개의 클러스터가 존재할수도 있음
1. **Node**
    - 하나의 단위 프로세스
    - 역할에 따라 Master-eligible, Data, Ingest, Tribe 노드로 구분
1. **Index**
    - RDBMS에서 database와 대응하는 개념
1. **Shard**
    - 스케일 아웃을 위해 index를 여러 shard로 쪼갬
<br>

## Logstash
데이터 수집 파이프라인 도구
<br><br>
데이터베이스의 데이터, 로우 데이터, 윈도우 이벤트 등으로부터 데이터를 수집한다. 
입출력 도구이며, Input > filter > output 의 pipeline 구조로 구성되어 있다.

1. **Input**
    - Beats, CloudWatch, Eventlog 등의 다양한 입력을 지원하여 데이터를 수집
    - file, syslog(RFC3164형식), beats(filebeats)

1. **Filter**
    - 형식이나 복잡성에 상관없이 설정을 통해 데이터를 동적으로 변환
    - grok, mutate, drop, clone, geoip

1. **Output**
    - ElasticSearch, Email, ECS, Kafka등 원하는 저장소에 데이터를 전송

<br>

## Kibana
Elasticsearch에서 색인된 데이터를 검색하고 시각화하는 기능을 제공한다. Elastic Stack 클러스터를 모니터링, 관리 및 보호하기 위한 사용자 인터페이스의 역할과 Elastic Stack에서 개발된 기본 제공 솔루션의 중앙 집중식 허브 역할도 한다.

### 기능 메뉴
1. Discover
    - Elasticsearch에 저장된 데이터를 한눈에 확인할 수 있는 메인 페이지
    - Auto-refresh
        + Elasticsearch의 검색은 하나의 Thread로 동작
        + 주기를 짧게 하면 많은 request 발생
        + Dashboard에 여러개의 모듈이 있으면 각각의 search request 발생
        + 실시간 모니터링을 하려면 Dashboard 차트 수를 줄이거나 주기를 너무 짧지 않게 해야 함

1. Visualize
    - Elasticsearch에 수집된 결과를 시각화하여 표현
    - 영역 차트, 데이터 테이블, 선형 차트, 마크다운 위젯, 메트릭, 원형 차트, 타일 맵, 세로 막대 차트

1. Dashboard
    - Visualize를 통해 시각화한 객체를 모아 하나의 Dashboard에 배치하여 한눈에 확인할 수 있도록 함

<br>

## Beats
경량 에이전트로 설치되어 데이터를 Logstash 또는 ElasticSearch로 전송함.Filebeat, Meticbeat 등이 있으며 Libbeat을 이용하여 직접 구축 가능.(Golang 짱짱)

- **FileBeat** : 서버에서 로그파일을 제공
- **PacketBeat** : 응용 프로그램 서버간에 교환되는 트랜잭션에 대한 정보를 제공
- **MetricBeat** : 운영 체제 및 서비스에서 Metrics를 주기적으로 수집

