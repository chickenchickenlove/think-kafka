# think-kafka


# 문제
- `KafkaStreams`의 `StateStore`의 오래되었지만 더는 업데이트 되지 않은 것들은 다음 문제를 야기할 수 있음.
  - 오래된 `ChangeLog`가 계속 Join되면서 불필요한 로그를 더욱 많이 만들어낼 수 있음. (Left - Right Join)
  - 오래된 `ChangeLog`가 계속 Memory, Disk의 용량을 차지하고 있을 수 있음.
  - 오래된 `ChangeLog`는 더 많은 `Restore` 시간을 요구함.
  - 사용자는 인지하지 못하고 부정확한 데이터가 계속 나올 수 있음.


## `Ignore oldest log which should be restored with RestoreConsumer`

#### Glosary
- Group A : Limit 시간을 지난 녀석들
- Group B : Limit 시간을 지나지 않은 녀석들.


#### Case1 : 1 `streams`, 1 `topic`, 1 `partitions`.
- 즉, 1 `streams` = 1 `partitions`.
- Case1-1 : Local `RocksDB` 있는 상태 (문제 없음) → 같은 머신에서 재기동
  - Local `RocksDB`에는 상태가 그대로 남아 있을 수도 있음.
  - 궁극적으로는 `ChangeLog` == `RocksDB`가 될 수 있음. 
- Case1-2 : Local `RocksDB` 없는 상태 (문제 없음) → 다른 머신에서 재기동
  - 괜찮음.
 
#### Case2 : 1 `streams`, 1 `topic`, 10 `partitions`
- Case2-1 : Local `RocksDB` 있는 상태 (문제 없음) → 같은 머신에서 재기동
  - Case 1-1과 동일함.
- Case2-2 : Local `RocksDB` 없는 상태 (문제 없음) → 다른 머신에서 재기동
  - Case 1-2와 동일함.
 
#### Case3 : 10 `streams`, 1 `topic`, 1 `partitions`
- Case3-1 : Local `RocksDB` 있는 상태 → 같은 머신에서 재기동
  - Case 1-1과 동일함.
- Case3-2 : Local → 다른 머신에서 재기동
  - Case 1-2과 동일함.
 
#### Case4 : 1 `streams`, 1 `topic`, 10 `partitions`.
- Case4-1 : Local `RocksDB` 있는 상태 → 같은 머신에서 재기동
  - Case 1-1과 동일함.
- Case4-2 : Local → 다른 머신에서 재기동
  - Case 1-2과 동일함.


#### Case5 : 2 `streams`, 2 `topic`, 2 `partitions`
- Case5-1 : `stream` 1개 같은 머신에서 재기동
  - 남아있는 `Stream` + 기존 `Stream`은 오래된 로그 상태를 가지고 있음.
- Case5-2 : `stream` 1개 다른 머신에서 재기동
  - 남아있는 `Stream`은 오래된 로그 상태를 가지고 있음. 새로 기동된 녀석은 오래된 로그 상태 없음. 따라서 서로 다른 상태 → 문제 발생 가능성 존재함.
- Case5-3 : 2개 재시작. 서로 다른 머신에서 재기동
  - 둘다 모두 오래된 로그를 가지고 있지 않음. 문제 없음.
- Case5-4 : 2개 재시작. 한 개는 이전 머신,  한 개는 새로운 머신에서 재기동
  - Case5-2와 동일한 상태가 됨. 


### 파티션별 / 토픽별로 서로 다른 상태(시점 관점)는 괜찮은가?
- 잠재적인 문제를 가지고 있음.
  1. 리파티션 시, 한쪽 파티션은 오래된 데이터까지, 한쪽 파티션은 최신 데이터만 가지고 있다면 문제일 수 있음.
  2. 조인 시, 한쪽 토픽은 오래된 데이터, 한쪽 토픽은 최신 데이터만 있다면 문제일 수 있음.
- 오래된 데이터 / 최신 데이터가 서로 다른 State를 가지고 있는 경우에는 잠재적인 데이터 문제가 발생할 수 있음. 


### 파티션 / 토픽별로 서로 다른 상태를 맞추는 방법은?
- 하나의 리밸런싱이 발생하면, `Kafka Streams Cluster` 전체가 재기동해서 `Restore` 함.
  - 장점 : `Kafka Streams Cluster` 전체의 `State`의 최신 상태를 달성할 수 있음.
  - 단점 : `Stop the world` 시간이 길어짐. (전체 `Kafka Streams Application`의 `ChangeLog`성 토픽을 `Log Replay` 해야함)

### 결론 
- `RestoreConsumer`를 이용한 `Old Log Filter`는 `Kafka Streams Cluster`


## RocksDB에 저장할 때, 레코드의 생성 시점을 같이 저장

### Psuedo Code
```java
// When Input Case
final Object key = record.key();
final Object value = record.value();
final long createdTime = System.currentTimeMillis();

final RocksDBValueEntry rocksDBValue RocksDBValueEntry.create(value, createdTime);
rocksDB.put(key, rocksDBValue);
```

```java
// When output Case
final long interval = this.processorContext.getExpiredTime();
final long now = System.currentTimeMillis();
final long expiredThresholdTime = now - interval;


rocksDBEntry.stream().
            .filter(rocksDBEntry -> rocksDBEntry.getCreateTime() < expiredThresholdTime)
            .forEach(rocksDBEntry -> proccesor.forward(record, rocksDBEntry);
```

- RocksDB는 객체를 저런 방식으로도 저장할 수 있는지?
- RocksDB에 저장된 ChangeLog는 이런 형식으로 모든 Kafka Topic에 저장될 것임. 따라서 Kafka Streams Cluster 내부에서의 State는 동일하게 유지될 수 있을 것임. 







