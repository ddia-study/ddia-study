# Chapter 8. 분산 시스템의 골칫거리

***분산 시스템을 다루는 것은 뭔가 잘못될 수 있는 새롭고 흥미진진한 방법이 많다는 점이다.***

***결국 엔지니어로서의 우리의 임무는 모든 게 잘못되더라도 제 역할을 해내는 시스템을 구축하는 것이다.***

## 결함과 부분 장애

**부분 장애(partial failure)** : 분산 시스템에서는 시스템의 어떤 부분은 잘 동작하지만 다른 부분은 예측할 수 없는 방식으로 고장날 수 있다.

부분 장애는 **비결정적**이다. 

### 클라우드 컴퓨팅과 슈퍼 컴퓨팅

- HPC : 분산 시스템보다는 단일 노드 컴퓨터에 가까움
- Cloud Computing

분산 시스템에서 의심, 비관주의, 편집증은 그 값어치를 한다. 

## 신뢰성 없는 네트워크

분산 시스템은 비공유 시스템, 즉 네트워크로 연결된 다수의 장비다.

네트워크는 이 장비들이 통신하는 유일한 수단이다.

**비공유의 장점?**

- 상대적으로 저렴
- 상품화된 클라우드 서비스 사용 가능
- 지리적으로 분산된 여러 DC에 중복 배치하여 신뢰성 확보 가능

**Q. synchronous (packet) network?**

- duplex
- 보통 전용 회로 필요하다고 함
- 빠름

어쨌거나, 비동기 네트워크에서는 응답을 받지 못한 경우 이유를 아는 것은 불가능하다.

이런 문제를 다루는 방법 : Timeout

### 현실의 네트워크 결함

**네트워크 분단(network partition, netsplit)**

네트워크 결함 때문에 네트워크 일부가 다른 쪽과 차단되는 것

반드시 네트워크 결함을 견뎌내도록(tolerating) 처리할 필요는 없다.

그러나 소프트웨어가 네트워크 문제에 어떻게 반응하는지 알고 시스템이 그로부터 복구할 수 있도록 보장해야 한다 

### 결함 감지

- LB는 죽은 노드로 요청을 그만 보내야 한다.
- 단일 리더 복제를 사용하는 분산 DB에서 리더에 장애가 나면 팔로워 중 하나가 리더로 승격해야 한다.

네트워크에 대한 불확실성 때문에 결함을 감지하기 어려울 수 있다.

그렇기에, 보내는 쪽과 수신하는 쪽 모두 처리가 필요하다.

### 타임아웃과 기약 없는 지연(unbounded delay)

타임아웃이 길면 노드가 죽었다고 선언될 때까지 기다리는 시간이 길어진다.

타임아웃이 짧으면 결함을 빨리 발견하지만 노드가 일시적으로 느려졌을 뿐인데도 죽었다고 잘못 선언할 위험이 높아진다.

노드가 죽으면 해당 부하가 다른 노드에 쏠리게 되므로, 시스템이 높은 부하에 허덕이는 중이라면 성급하게 노드가 죽었다고 선언하는 것은 좋은 방법이 아니다.

비동기 네트워크는 **unbounded delay**가 있다.

### Network congestion과 큐 대기


- network congestion : 네트워크 링크가 붐비면 패킷은 슬롯을 얻을 수 있을 때까지 기다려야 한다. 데이터가 많이 들어와 큐가 꽉 차있다면? 패킷이 유실될 수 있다.
- 패킷이 목적지에 도착했을 때 CPU가 바쁘면 운영체제에서 큐에 넣어둔다.
- 가상 환경에서 실행되는 운영체제는 다른 장비가 CPU 코어를 사용하는 동안 멈출 수도 있다. 그 동안 받은 패킷을 가상 장비 모니터가 큐에 넣어서 버퍼링한다. 
- TCP는 flow control을 수행한다. congestion avoidance, backpressure를 통해 수신 노드나 링크에 과부하를 가하지 않도록 송신율을 제한한다. 즉, 네트워크로 들어가기 전에도 큐 대기를 할 수 있다.


신뢰성과 지연 변동성 사이에 트레이드 오프 관계가 있다.

공유된 자원을 다른 사용자가 사용하는 것을 제어하거나 간파할 수 없는 경우, **시끄러운 이웃**이 가까이 있다면 네트워크 지연 변동이 클 수 있다.

타임아웃을 자동으로 조절할 수도 있다.

### 동기 네트워크 vs 비동기 네트워크

동기 네트워크 예시 : 전화 회선

여러 라우터를 거쳐도 큐 대기 문제를 겪지 않고, 네트워크 end-to-end 지연 시간 최대치가 고정돼 있다.

이를 **bounded delay**라고 한다.

### 그냥 네트워크 지연을 예측 가능하게 만들 수는 없을까?

대체 왜 DC Network와 인터넷은 패킷 교환을 사용할까?

이들은 **bursty traffic**에 최적화됐기 때문이다.

QoS(패킷에 우선순위를 매기고 스케줄링함)과 진입 제어(admission control, 전송 측에서 전송률을 제한)를 잘 쓰면 패킷 네트워크에서 회선 교환을 흉내낼 수도 있다.

---

## 신뢰성 없는 시계

분산 시스템에서는 통신이 즉각적이지 않으므로 시간을 다루기 까다롭다.

게다가 네트워크에 있는 개별 장비는 자신의 시계를 갖고 있다.

Network Time Protocol, NTP를 통해 컴퓨터 시계를 조정할 수도 있다.

### time-of-day clock

어떤 달력에 따라 현재 날짜와 시간을 반환한다.

NTP로 동기화 되는데, 로컬 시계가 NTP 서버보다 너무 앞서면 강제로 리셋되어 과거 시점으로 거꾸로 뛰는 것처럼 보일 수 있다.


### monotonic clock

타임아웃이나 서비스 응답 시간 같은 지속 시간(시간 구간)을 재는 데 적합하다.

단조 시계는 늘 앞으로 흐른다.

시계의 절대적인 값은 의미가 없으며, 서로 다른 두 대의 컴퓨터에서 나온 단조 시계 값을 비교하는 것은 의미가 없다.

**얘기할만 한 내용**

트위터는 timestamp, 특히 epoch time을 이용해서 UUID를 만든다고 한다.

epoch time이면 time-of-day clock일텐데 어떻게 동기화를 할까?

> You should use NTP to keep your system clock accurate. Snowflake protects from non-monotonic clocks, i.e. clocks that run backwards. If your clock is running fast and NTP tells it to repeat a few milliseconds, snowflake will refuse to generate ids until a time that is after the last time we generated an id. Even better, run in a mode where ntp won't move the clock backwards. See http://wiki.dovecot.org/TimeMovedBackwards#Time_synchronization for tips on how to do this. - 실제 구현한 github에 적혀 있는 내용

Twitter snowflake : https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake

실제 구현 : https://github.com/twitter-archive/snowflake/tree/updated_deps

**Q. 해상도?**

### 시계 동기화와 정확도

하드웨어 시계와 NTP는 변덕스러울 수 있다.

- 장비의 온도에 따라 drift(빠르거나 느리게 실행되는) 현상이 발생할 수 있다.
- 컴퓨터 시계와 NTP의 차이가 크면 동기화가 거부되거나 로컬 시계가 리셋될 수도 있다.
- 노드와 NTP 사이가 방화벽으로 막히면 잘못된 설정이 얼마동안 알려지지 않을 수도 있다.
- NTP 동기화는 잘해야 네트워크 지연만큼만 좋을 수 있다.
- NTP 서버 자체가 잘못될 수도 있다.
- 윤초가 발생하면, 윤초를 고려하지 않은 시스템이 고장낼 수도 있다. 최선의 방법은 윤초 조정을 서서히 수행하게 하여 NTP 서버가 거짓말을 하게 하는 것일 수 있다.

### 동기화된 시계에 의존하기

대부분의 시간에 아주 잘 동작하지만 견고한 소프트웨어는 잘못된 시계에 대비할 필요가 있다.

한 가지 문제는 시계가 잘못된다는 것을 눈치채지 못하기 쉽다는 것이다.

따라서 동기화된 시계가 필요한 소프트웨어를 사용한다면 필수적으로 모든 장비 사이의 시계 차이를 조심스럽게 모니터링해야 한다. 

### 이벤트 순서화용 타임스탬프

LWW로 충돌을 해소할 때, 잘못된 타임스탬프는 데이터를 손실 시킬 수 있다. 

따라서 가장 최근 값을 유지하고 다른 것들을 버림으로써 충돌을 해소하고 싶은 유혹이 들더라도 "최근"의 정의는 로컬 일 기준 시계에 의존하며 그 시계는 틀릴 수도 있다는 것을 아는 게 중요하다.

잘못된 순서화가 발생하지 않을 정도로 NTP 동기화를 정확하게 하는 것은 어려울 것이다.

이른바 **Logical clock**은 증가하는 카운터를 기반으로 하며 이벤트 순서화의 안전한 대안이다. 즉, 상대적인 순서만 측정한다.

### 시계 읽기는 신뢰 구간이 있다.

시계 읽기를 어떤 시점으로 생각하는 것은 타당하지 않다.

어떤 신뢰 구간에 속하는 시간의 범위로 읽는 게 나을 것이다.

**spanner에 있는 google TrueTime API** : 예외적으로 신뢰 구간 보여줌

5~6pg에 TrueTime 관련 내용이 존재 : https://static.googleusercontent.com/media/research.google.com/ko//archive/spanner-osdi2012.pdf

### 전역 스냅숏용 동기화된 시계

여러 DC에 있는 여러 장비에 분산되어 있는 DB라면 전역 단조 증가 트랜잭션 ID를 생성하기 어렵다.

스패너는 시계 신뢰 구간을 이용하여 타임스탬프를 트랜잭션 ID로 쓴다.

두 개의 트랜잭션이 있다고 할 떄, 구간이 겹치지 않으면 인과성을 판단할 수 있다.

겹치는 경우에만 실행 순서를 확신할 수 없는데, 이를 위해 읽기 쓰기 트랜잭션을 커밋하기 전에 의도적으로 신뢰 구간의 길이만큼 기다린다.

그렇게 하면 그 데이터를 읽을지도 모르는 트랜잭션은 충분히 나중에 실행되는 게 보장되므로 신뢰 구간이 겹치지 않는다. 

### 프로세스 중단

프로그램 실행 중 예상치 못한 중단이 있었다면, 중단 이후로 권한이 없지만 프로세스가 뭔가 안전하지 않은 일을 한 상태일지 모른다.(lease 예시)

아주 오랫동안 멈추는 경우는 다양하다.

- stop-the-world
- 가상 장비 suspend & resume
- 노트북 덮개 닫기
- context switching (steal time)
- I/O 연산
- Swap to disk (thrashing)
- SIGSTOP

단일 장비에서는 뮤텍스, 세마포어 등 다양한 도구로 thread-safe를 구현하지만, 분산 시스템에서는 바로 변형해서 사용할 수 없다.

분산 시스템의 노드는 어느 시점에 실행이 상당한 시간 동안 멈출 수 있다고 가정해야 한다.

### 응답 시간 보장

스레드와 프로세스는 unbounded delay가 일어날 수 있지만 노력하면 중단의 원인을 제거할 수 있다.

mission-critical한 소프트웨어는 응답해야 하는 deadline이 명시되고, 이를 만족하지 못하면 장애가 발생될 수 있다.

**mission critical?** : https://en.wikipedia.org/wiki/Mission_critical

이런 시스템을 hard real-time system이라고 한다.

실시간 보장을 제공하려면, 프로세스가 명시된 간격의 CPU 시간을 할당받을 수 있게 보장되도록 스케줄링해 주는 RTOS가 필요하다.

실시간 시스템은 매우 많은 비용이 들고, 고성능이 아닐 수도 있으며, 언어 및 도구가 제한된다.

대부분 서버 시스템에게 실시간 보장은 전혀 경제적이지도, 적절하지도 않기 때문에 중단과 시계 불안정으로부터 고통받을 수 밖에 없다.

### 가비지 컬렉션의 영향을 제한하기

GC 중단을 노드가 잠시동안 계획적으로 중단되는 것으로 간주하고 노드가 GC 하는 동안 클라이언트로부터 요청을 다른 노드들이 처리하게 하는 것이다.

이런 트릭은 GC 중단을 클라이언트로부터 감추고 응답 시간의 상위 백분위를 줄여준다.

이 아이디어의 변종은 컬렉션을 빨리 할 수 있는 수명이 짧은 객체만 가비지 컬렉터를 사용하고 수명이 긴 객체의 전체 GC가 필요할 만큼 객체가 쌓이기 전에 주기적으로 프로세스를 재시작하는 방법이다. 

> GC에 대해서 알아보기 전에 알아야 할 용어가 있다. 바로 'stop-the-world'이다. stop-the-world란, GC을 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것이다. stop-the-world가 발생하면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다. GC 작업을 완료한 이후에야 중단했던 작업을 다시 시작한다. 어떤 GC 알고리즘을 사용하더라도 stop-the-world는 발생한다. 대개의 경우 GC 튜닝이란 이 stop-the-world 시간을 줄이는 것이다. - naver D2 블로그 글 중

GC 관련 Naver D2 글 : https://d2.naver.com/helloworld/1329