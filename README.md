# think-kafka


### Ignore oldest log which should be restored with RestoreConsumer

#### Glosary
- Group A : Limit 시간을 지난 녀석들
- Group B : Limit 시간을 지나지 않은 녀석들.


#### Case1 : 1 streams, 1topic, 1 partitions.
- 즉, 1 streams = 1 partitions.
- Case1-1 : Local RocksDB 있는 상태 (문제 없음) → 같은 머신에서 재기동
  - Local RocksDB에는 상태가 그대로 남아 있을 수도 있음.
  - 궁극적으로는 ChangeLog == RocksDB가 될 수 있음. 
- Case1-2 : Local RocksDB 없는 상태 (문제 없음) → 다른 머신에서 재기동
  - 괜찮음.
 
#### Case2 : 1 streams, 1topic, 10 partitions
- Case2-1 : Local RocksDB 있는 상태 (문제 없음) → 같은 머신에서 재기동
  - Case 1-1과 동일함.
- Case2-2 : Local RocksDB 없는 상태 (문제 없음) → 다른 머신에서 재기동
  - Case 1-2와 동일함.
 
#### Case3 : 10 streams, 1topic, 1 partitions
- Case3-1 : Local RocksDB 있는 상태 → 같은 머신에서 재기동
  - Case 1-1과 동일함.
- Case3-2 : Local → 다른 머신에서 재기동
  - Case 1-2과 동일함.
 
#### Case4 : 1 streams, 1topic, 10 partitions.
- Case4-1 : Local RocksDB 있는 상태 → 같은 머신에서 재기동
  - Case 1-1과 동일함.
- Case4-2 : Local → 다른 머신에서 재기동
  - Case 1-2과 동일함.


#### Case5 : 2 streams, 2topic, 2 partitions
- Case5-1 : stream 1개 같은 머신에서 재기동
  - 남아있는 Stream + 기존 Stream은 오래된 로그 상태를 가지고 있음.
- Case5-2 : stream 1개 다른 머신에서 재기동
  - 남아있는 Stream은 오래된 로그 상태를 가지고 있음. 새로 기동된 녀석은 오래된 로그 상태 없음. 따라서 서로 다른 상태 → 문제 발생 가능성 존재함.
- Case5-3 : 2개 재시작. 서로 다른 머신에서 재기동
  - 둘다 모두 오래된 로그를 가지고 있지 않음. 문제 없음.
- Case5-4 : 2개 재시작. 한 개는 이전 머신,  한 개는 새로운 머신에서 재기동
  - Case5-2와 동일한 상태가 됨. 


## 파티션별로 서로 다른 상태는 괜찮은가?
- 잠재적인 문제를 가지고 있음.
  1. 리파티션 시, 한쪽 파티션은 오래된 데이터까지, 한쪽 파티션은 최신 데이터만 가지고 있다면 문제일 수 있음.
  2. 조인 시, 한쪽 토픽은 오래된 데이터, 한쪽 토픽은 최신 데이터만 있다면 문제일 수 있음.
- 오래된 데이터 / 최신 데이터가 서로 다른 State를 가지고 있는 경우에는 잠재적인 데이터 문제가 발생할 수 있음. 






