# Elasticsearch

https://aspell.tistory.com/75
https://esbook.kimjmin.net/06-text-analysis/6.1-indexing-data

## Elasticsearch 란?
1. Full-Text 검색 엔진
    - 대량의 데이터를 검색하기 위한 별도의 Index를 만드는 기술

1. 분석 엔진

1. 준 실시간 분석
    - 성능을 위해 데이터를 메모리에 유지
        + 노드가 죽으면 데이터 유실 가능
    - Split brain
        + 클러스터로 구성된 두 시스템 그룹간 네트워크의 일시적 동시 단절현상이 발생 시 나타나는 현상
        + 클러스터 상의 모든 노드들은 노드 각자가 자신을 primary(Master)라고 인식하게 되는 상황

## Inverted Index
#### 관계형 데이터베이스
| ID | Text |
|:---|:-----|
|doc1|The quick brown fox|
|doc2|The quick brown fox jumps over the lazy dog|
|doc3|The quick brown fox jumps over the quick dog|
|doc4|Brown fox brown dog|
|doc5|Lazy jumping dog|

- 'fox'가 들어간 문서를 찾기 위해 doc1 ~ doc5를 모두 검색

#### Elasticsearch
| Term | ID | Term | ID |
|:-----|:---|:-----|:---|
|The|doc1, doc2, doc3|quick|doc1, doc2, doc3|
|brown|doc1, doc2, doc3, doc4|fox|doc1, doc2, doc3, doc4|
|jumps|doc2, doc3|over|doc2, doc3|
|the|doc2, doc3|lazy|doc2|
|dog|doc2, doc3, doc4, doc5|Brown|doc4|
|Lazy|doc5|jumping|doc5|

- 추출된 각 키워드를 텀(Term) 이라 함
- fox를 포함하고 있는 문서를 바로 찾을 수 있음


## 구조
![Optional Text](images/es_head.png)
1. Cluster
    - {-Vxovpx, 71_q51B, Krvd6u4, oV7t_ZK, qiQbIvH, uLdsPJ4} 집합 
1. Node
    - 마스터 노드 : -Vxovpx, 71_q51B, Krvd6u4
    - 데이터 노드 : oV7t_ZK, qiQbIvH, uLdsPJ4

1. Index
    - test, kibana_sample_data_flights, .ben-lectes-kibana
    - 하나의 인덱스가 여러개의 데이터 노드에 저장

1. Document
    - test : 0, 1, 2, 3, 4
    - kibana_sample_data_flights : 0
    - .ben-lectes-kibana : 0

1. Shard
    - 원본 샤드 : test Index에서 oV7t_ZK Node의 1
    - 복제본 샤드 : test Index에서 qiQbIvH Node의 1
    - 원본 샤드가 유실되면 복제본 샤드가 원본 샤드로 승격
    - 원본 샤드에 대한 복제본 샤드가 다른 노드에 저장되어야 함
        + kibana_sample_data_flights의 경우  qiQbIvH 노드가 죽으면 0번 샤드를 복구할 수 없음

## 쿼리
| 관계형 | ES |
|:--------|:--------|
| select | GET |
| insert | POST |
| update | PUT |
| delete | DELETE |

1. Select
    - curl -XGET localhost:9200/index_name/doc_type/1
    - select * from doc_type where id = 1

1. Insert
    - curl -XPOST localhost:9200/index_name/doc_type/1 -d '{xxx}'
    - insert into index_name values {xxx}

1. Update
    - curl -XPUT localhost:9200/index_name/doc_type/1 -d '{xxx}'
    - update index_name set xxx where id = 1

1. Delete
    - curl -XDELETE localhost:9200/index_name/doc_type/1
    - delete from index_name where id = 1