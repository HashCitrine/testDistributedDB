# testIMDG
## IMDG (In-Memory Data Grid)
![IMDG](https://image.samsungsds.com/kr/insights/memory1.jpg?queryString=20231030024258)
- Redis와 같이 데이터를 Disk가 아닌 `In-Memory`에 저장하며, 이에 `Data Grid` 구조를 이용해 분산 환경에 적용 가능하도록 만들어진 `분산 메모리 시스템`
- In-Memory의 빠른 데이터 처리 속도와 분산 환경의 고가용성, 확장성의 장점을 가짐

## Hazelcast
- 오픈 소스 IDMG
- 각 Cluster Member가 `Master - Slave`가 아닌 `Peer-to-Peer`의 관계를 가짐
- ` SQL-like` 구문 사용 가능
- `DistributedMultiMap`, `Distributed Collections` 자료형 지원
- `DistributedTopic` & `DistributedEvent` 기능을 통해 Message Queue처럼 이용 가능
- 

### [docker-compose.yml](https://github.com/HashCitrine/testIMDG/tree/master/hazelcast/docker-compose.yml)
- Hazelcast를 Docker Container로 복수 구성하여 테스트

```yaml
services:
  hazelcast-manager:
    image: hazelcast/management-center:latest-snapshot
    ports:
      - "8080:8080"
    networks:
      - hazelcast-network
  hazelcast-imdg-1:
    image: hazelcast/hazelcast:5.3.6
    ports:
      - "5701:5701"
    environment:
      HZ_NETWORK_PUBLICADDRESS: {HOST_IP}:5701
      HZ_CLUSTERNAME: hello-world
    networks:
      - hazelcast-network
  hazelcast-imdg-2:
    ports:
      - "5702:5702"
    # 'ports' 이외 옵션은 'hazelcast-imdg-1'와 동일
  hazelcast-imdg-3:
    ports:
      - "5703:5703"
    # 'ports' 이외 옵션은 'hazelcast-imdg-1'와 동일
    
networks:
  hazelcast-network:
    driver: bridge
```

### Example Test
``` shell
$ hz-cli --targets hello-world@{host_ip}:5701 sql
Connected to Hazelcast 5.3.6 at [{host_ip}]:5701 (+2 more)  # 같은 Network의 5702, 5703 포트를 이용하는 Hazelcast가 자동으로 인식된 상태
Type 'help' for instructions
sql> CREATE MAPPING my_distributed_map TYPE IMap OPTIONS ('keyFormat'='varchar','valueFormat'='varchar');
OK
sql> SINK INTO my_distributed_map VALUES
   > ('1', 'John'),
   > ('2', 'Mary'),
   > ('3', 'Jane');
OK
sql> SELECT * FROM my_distributed_map;
+--------------------+--------------------+
|__key               |this                |
+--------------------+--------------------+
|3                   |Jane                |
|1                   |John                |
|2                   |Mary                |
+--------------------+--------------------+
3 row(s) selected
```

## 참조
- [시스템 성능 개선을 위한 In-Memory 기술 활용 ' In-Memory Data Grid 활용 사례 '](https://www.samsungsds.com/kr/insights/in-memory-data-grid.html)
- [[Hazelcast] About Hazelcast](https://medium.com/aisland/hazelcast-about-hazelcast-94a30838c0c)
- [Start a Local Cluster in Docker](https://docs.hazelcast.com/hazelcast/5.3/getting-started/get-started-docker)
- [[기록은 재산이다] Hazelcast 공유](https://supawer0728.github.io/2018/03/11/hazelcast/)
- [이제 필요한 것은 In Memory Data Grid](https://d2.naver.com/helloworld/106824)