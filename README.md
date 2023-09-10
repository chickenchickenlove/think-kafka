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







