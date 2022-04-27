# 6장 파티셔닝
- 샤딩, 파티셔닝
- 목표 : 확장성
  - 트랜잭션 작업부하용, 분석용

## 파티셔닝과 복제
- 한 노드에 여러 파티션을 저장할 수 있다.
- 각 노드는 어떤 파티션에게는 리더이면서 다른 파티션에게는 팔로워가 될 수 있다.

## 키-값 데이터 파티셔닝
- skewed : 쏠림
  - 특정 파티션이 데이터가 다른 파티션에 비해 데이터가 많거나 질의를 많이 받는다면 문제가 발생하게 된다.(속도 저하 등)
  - 핫스팟 : 쏠림 현상이 일어난 노드

### 키 범위 기준 파티셔닝
- 각 파티션에 연속된 범위를 할당하는 방법
- 장점 : 범위 스캔이 쉬워진다.
- 단점 : 핫스팟이 유발된다.(Skew 현상이 일어난다.)
  - 단점을 극복하기 위해서는 어플리케이션 레벨 분석을 통해 핫스팟을 해결해야한다. 대신 장점을 잃을 수도 있다.

### 키의 해시값 기준 파티셔닝
- 많은 분산 데이터스토어에서 사용하는 방법
- 장점 : 핫스팟이 생기지 않는다.(좋은 해쉬함수를 사용하면)
  - 일관성 해쉬
    - CDN(content delivery network)에서도 널리 사용하는 방법
    - 용어 혼동을 피해서 일관성 해쉬(consistent hashing)보다는 해시 파티셔닝(hash partitioning)을 사용하는 것이 좋다.
    - 몽고DB : MD5, 볼드모트 : Fowler-Noll-Vo

### 쏠림 작업부하와 핫스팟 완화
- 현대 데이터 시스템은 자동 보정하기 어려우므로 어플리케이션에서 쏠림 완화해야한다.

- 참고자료:
  - 현실적인 (데이터 분석을 위한) BigData : Spark, redshift, bigQuery
  - [Spark 에서의 skew 현상과 성능](https://michaelheil.medium.com/understanding-common-performance-issues-in-apache-spark-deep-dive-data-skew-e962909f3d07)
  - [AWS redshift vs GCP bigQuery](https://thewayitwas.tistory.com/457)
  - [RDB에서의 파티셔닝에서 알아야되는 개념 - 파티션 프루닝 (partition pruning), 복합 파티션, 서브파티션 (composite partition, sub partition)](https://hoing.io/archives/7909)

---

## 파티셔닝과 보조 색인
- 키-값 저장소에서는 보조 색인이 복잡하다.
- 관계형 데이터베이스에서는 보조 색인이 핵심 요소이며, 문서 데이터베이스에서도 흔하다.

### 문서 기준 보조 색인 파티셔닝
- 문서 파티셔닝 색인 : 지역 색인(local index)
- 스캐터(scatter)/개더(gather) : 모든 파티션으로 질의를 보내고 이를 정돈하여 처리하는 방식
  - 꼬리 지연(tail latency)이 시간 증폭(time amplification)를 가져오기 쉽다.

### 용어 기준 보조 색인 파티셔닝
- 전역 색인 : 모든 파티션의 데이터를 담당하는 색인
- 여러 노드에 분산시켜 저장한다.
- 일반적으로 용어 기준으로 파티셔닝한다.(term-partitioned)
- 현실적인 이유로 비동기로 갱신한다.

### 파티션 재균형화
- Q. 아래 항목에 대해서 답하세요.
  - 1. 질의 처리량이 증가할때 고려해야하는 자원은?
  - 2. 데이터셋 크기가 증가할때 고려해야하는 자원은?
  - 3. 장애를 대비하기 위해서 해야하는 작업은?
      ---
- 재균형화(rebalancing):
  - 재균형화 후, 부하(데이터 저장소, 읽기 쓰기 요청)가 클러스터 내에 있는 노드들 사이에 균등하게 분배돼야 한다.
  - 재균형화 도중에도 데이터베이스는 읽기 쓰기 요청을 받아들여야 한다.
  - 재균형화가 빨리 실행되고 네트워크와 디스크 I/O 부하를 최소화할 수 있도록 노드들 사이에 데이터가 필요 이상으로 옮겨져서는 안된다.

#### 재균형화 전략
- 안좋은 예시 : 해시값에 mod 연산
  - Q. 왜 쓰면 안될까요?

##### 파티션 개수 고정(정적 파티셔닝)
- 장점 : 단순하다. 성능에 따른 부하 분산이 용이하다.
- 단점 : 파티션 개수 결정이 어렵다. 데이터셋 크기 예측이 어려울 경우 적합하지 않다.

- Riak(리악), ElasticSearch(엘라스틱서치), CouchBase(카우치베이스), VoltDB(볼드모트)

##### 동적 파티셔닝
- 장점 : 정적 파티셔닝의 단점
- 단점 : 어렵다. 반드시 정적보다 좋다는 보장은 없다.

- HBase, RethinkDB(리씽크 디비)

##### 노드 비례 파티셔닝
- 장점 : 동적파티셔닝의 장점을 현실적으로 가져옴
- 단점 : 현실적으로 가져옴

- Cassandra(카산드라), Ketama(케타마)

### 요청 라우팅
- 기본적인 3가지 방식:
  - 아무 노드나 접속 후, 올바른 노드를 찾아가는 방식 : 카산드라와 리악
  - 라우팅 계층을 별도로 만들고 라우팅 계층에서 이를 처리하는 방식 : 주키퍼
  - 클라이언트가 파티셔닝에 대한 정보를 알고 있어 바로 접속하는 방식

- [가십 프로토콜](https://en.wikipedia.org/wiki/Gossip_protocol)
  - 카산드라와 리악에서 사용

- 카우치 베이스에서는 목시라는 라우팅 계층을 사용하여 이를 처리한다.

- 참고자료:
  - [HBase의 개념](https://cyberx.tistory.com/164)
  - [HBase와 Zookeeper와의 관계](https://sungwookkang.com/1363)
  - 현실적으로 우리가 파티셔닝을 사용하기 위해서는 어떻게 해야할까?
    - 어플리케이션 개발자의 관점, DevOps의 관점, 데이터 분석가의 관점
    - 실시간 어플리케이션일떄와 아닐때에 대해서 답변이 달라지는가?

#### 병렬 질의 처리
- 대부분 위에서 나온 방법으로 처리가 됨
- MPP(massively parallel processing) : 대규모 병렬 처리
  - 10장에서 추가적으로 다룸

      ---
# 7장 트랜잭션
- safety guarantee : 안정성 보장
- 격리 수준:
  - 커밋 후 읽기
  - 스냅숏 격리
  - 직렬성

## 애매모호한 트랜잭션의 개념
- ACID : Atomicity, Consistency, Isolation, Durability
- BASE : Basically Available, Soft state, Eventual consistency

### 원자성
- commit, abort

- 참고자료 :
  - [Transaction에서 Commit과 Rollback은 비용이 다를까?](https://stackoverflow.com/questions/52484661/cost-performance-of-commit-vs-rollback-if-no-data-has-been-changed-in-the-tra)

### 일관성
- 현재 데이터베이스에서는 굉장히 모호한 단어가 되었다
  - 복제 일관성(replica consistency), 최종적 일관성(eventual consistency)
  - 일관성 해쉬
  - CAP에서의 선형성
  - 좋은 상태

### 격리성
- 참고자료 :
  - [MySQL에서의 실제 격리수준 설정하는 방법](http://labs.brandi.co.kr/2019/06/19/hansj.html)

### 지속성
- 한가지 방법만을 사용해서 현실에서는 절대적 보장을 제공할수 없다. 심지어 여러가지를 함께 사용하더라도 확률을 낮추는 것뿐이다.

## 단일 객체 연산과 다중 객체 연산
- RDB에서 `BEGIN TRANSACTION`, `COMMIT` 구문
- 비관계형 데이터베이스에서는 다중 put(multi-put) 연산을 제공할 수 있다.
  - [Redis 에서의 Transaction](https://minholee93.tistory.com/entry/Redis-Transaction)

### 단일 객체 쓰기
- read-modify-write 주기, compare-and-set 연산
  - 단일 객체에서 동시성 문제(대부분 갱신 손실 문제)를 방지하기 위해서 명심해야하는 수칙

### 다중 객체 트랜잭션의 필요성
- 관계형 데이터 모델에서 다른 테이블을 참조 할 경우
- 문서 데이터 모델에서 반드시 동시에 문서를 갱신해야할 경우
- 보조 색인이 있는 데이터베이스에서 값 변경시, 색인도 변경되어야하는 필요성

### 오류와 어보트 처리
- 오류 복구는 어플리케이션에게 있다
- 현실에서 발생하는 일들:
  - 트랜잭션이 성공했지만, 네트워크에서 커밋 성공을 알리는 메시지가 전송되지 않아, 트랜잭션이 두번 실행되는 경우 : 멱등성(idempotent)
  - 트랜잭션 재시도에 의해서 생기는 오류 과부화 현상 : exponential backoff, retry threshold
  - 일시적 오류가 아닌 영구적인 오류(제약 사항 위반 등) : 저장소와 어플리케이션 모델의 일치
  - 트랜잭션의 부수 효과(이메일 전송 등) : 배치 작업
  - 클라이언트 프로세스의 죽음 : 클라이언트 레벨 local db

- 참고자료:
  - Transaction 구현에도 표준이 있을까? (XA) : https://heni.tistory.com/10
  - 실제로 개발할때 트랜잭션은 어디서 구현되어 있을까?
    - spring(java) 기준으로는?
    - 다른 언어들은?
