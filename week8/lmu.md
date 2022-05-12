# 8장 분산시스템의 골칫거리
## 결함과 부분 장애
- 부분 장애(partial failure) : 분산 시스템에서 시스템의 어떤 부분은 잘 동작하지만 다른 부분은 예측할 수 없는 방식으로 고장 나는 것.

### 클라우드 컴퓨팅과 슈퍼컴퓨팅
- HPC(high-performance computing) : 수천 개의 CPU를 가진 슈퍼컴퓨터
  - 장애 발생시 연쇄적으로 죽게 한다.

- Cloud Computing

---
- 시스템이 커질 수록 항상 뭔가 고장난 상태라고 가정하는게 합리적이다.

## 신뢰성 없는 네트워크
- 요청이 손실됐을 수 있다.(누군가 네트워크 케이블을 뽑았다)
- 요청이 큐에서 대기하다가 나중에 전송될 수 있다.(네트워크나 수신자에 과부하가 걸렸을 수 있다).
- 원격 노드에 장애가 생겼을 수 있다(죽었거나 전원이 나갔을 수 있다.)
- 원격 노드가 일시적으로 응답하기를 멈췄지만(카비지 컬렉션 휴지(stop-the-world)가 길어졌을 수 있다.)
- 원격 노드가 요청을 처리했지만 응답이 네트워크에서 손실됐을 수 있다(네트워크 스위치의 설정이 잘못됐을 수 있다.)
- 원격 노드가 요청을 처리했지만 응답이 지연되다가 나중에 전송 될 수 있다(네트워크나 요청을 보낸 장비에 과부하가 걸렸을 수 있다)


- 위 모든 사항에 대해서 우리는 어떤 원인으로 인해서 문제가 발생했는지 알수가 없다.
- 대부분 타임아웃으로 처리한다.

### 현실의 네트워크 결함
- EC2 같은 공개 클라우드에서 일시적인 네트워크 결함이 자주 발생하는 것으로 유명하다.
- 반드시 네트워크 결함을 견뎌내도록 처리할 필요는 없다:
  - 하지만 고의로 네트워크 문제를 유발하고 시스템의 반응을 테스트하는 것은 일리가 있다.
  - 카오스 몽키

### 결함 감지
- 많은 시스템은 결함 있는 노드를 자동으로 감지할 수 있어야 한다:
  - 로드 밸런서는 죽은 노드로 요청을 그만 보내야 한다
  - 단일 리더 복제를 사용하는 분산 데이터베이스에서 리더에 장애가 나면 팔로워 중 하나가 리더로 승격돼야 한다

- 네트워크의 불확실성 때문에 노드가 동작 중인지 아닌지 구별하기 어렵다.:
  - 노드가 요청을 처리하다가 죽었다면 어디까지 처리하다가 죽었는지 알 방법이 없다.
  - 노드 프로세스가 죽었지만, 운영체제는 살아있다면, 빠르게 이어서 할 수 있다.(HBase 케이스)
  - 데이터센터 내 네트워크 스위치의 관리 인터페이스에 접근할 수있으면 하드웨어 수준의 링크 장애를 감지 할 수 있다.
  - ICMP 을 통해서 접속하려는 IP에 도달 가능한지 체크할 수는 있다.

### 타임아웃과 기약 없는 지연
- 타임아웃 길이에는 답이 없다.
- 기약 없는 지연(unbounded delay)이 가능하다.

### 네트워크 혼잡과 큐 대기
- 여러 다른 노드가 동시에 같은 목적지로 패킷을 보내려고 하면 네트워크 혼잡이 유발되고 재전송 해야한다.
  - [RED algorithm](https://www.mvps.net/docs/the-red-algorithm/#:~:text=The%20RED%20algorithm%20is%20also,blockage%20of%20the%20entire%20network.)
- 목적지 장비에 도착했을 때라도 모든 코어가 바쁜 상태라면 운영체제는 큐에 넣어둔다.
  - [Linux TCP Buffer Tuning](https://www.cyberciti.biz/faq/linux-tcp-tuning/)
  - [TCP, UDP Buffer](https://rocksea.tistory.com/64)
- vCPU가 수십 밀리초 동안 멈출 때가 흔하다.
- TCP의 흐름 제어(flow control)은 전체 부하를 낮추기 위해서 자신의 속도를 낮출 수 있다.

- 참고
  - [고성능을 위한 nginx 튜닝](https://couplewith.tistory.com/entry/%EA%BF%80%ED%8C%81-%EA%B3%A0%EC%84%B1%EB%8A%A5-Nginx%EB%A5%BC%EC%9C%84%ED%95%9C-%ED%8A%9C%EB%8B%9D-3-TCP-%EA%B4%80%EB%A0%A8-%EC%B2%98%EB%A6%AC%EB%9F%89-%EB%8A%98%EB%A6%AC%EA%B8%B0-%EB%A6%AC%EB%88%85%EC%8A%A4%EC%BB%A4%EB%84%90%ED%8A%9C%EB%8B%9D)
  - [네트워크 디자이너 기본기 쌓기 - 1 : 네트워크 개념과 동작 원리](https://zdnet.co.kr/view/?no=00000039134745)
  - [네트워크 디자이너 기본기 쌓기 - 2 : 큐잉 이론과 네트워크 모델링](https://zdnet.co.kr/view/?no=00000039135625)
  - [네트워크 디자이너 기본기 쌓기 - 3 : NW 시뮬레이션](https://zdnet.co.kr/view/?no=00000039136738)
  - [Awesome Prometheus alerts](https://awesome-prometheus-alerts.grep.to/rules.html)

## 동기 네트워크 대 비동기 네트워크
- 제한 있는 지연(bounded delay)

### 그냥 네트워크 지연을 예측 가능하게 만들 수는 없을까?
- bursty traffic
- QoS는 멀티 테넌트 데이터센터와 공개 클라우드에서는 사용할 수 없고 인터넷에서도 사용할 수 없다.

- 참고:
  - [KT 인터넷 속도 저하 사건](https://ko.wikipedia.org/wiki/KT_%EC%9D%B8%ED%84%B0%EB%84%B7_%EC%86%8D%EB%8F%84_%EC%A0%80%ED%95%98_%EC%82%AC%EA%B1%B4)
  - [화보로 이해하는 네트워크 장비 - QoS 전용 장비](https://zdnet.co.kr/view/?no=00000010052083)
  - [트래픽 폭증에도 안정을 유지한 카카오톡](https://brunch.co.kr/@kakao-it/170)
  - [LTTP: An LT-Code Based Transport Protocol for Many-to-One Communication in Data Centers](https://ieeexplore.ieee.org/document/6689483?tp=&arnumber=6689483&queryText%3DLTTP:%20An%20LT-Code%20Based%20Transport%20Protocol%20for%20Many-to-One%20Communication%20in%20Data%20Centers=)
  - [Scaling-out Ethernet for the Data Center (White Paper)](https://network.nvidia.com/pdf/whitepapers/WP-ethernet%20scaleout-WEB.pdf)

---
## 신뢰성 없는 시계
- 컴퓨터에 존재하는 다양한 어플리케이션은 시계에 의존한다.
  - 타임아웃, 응답시간, 시간당 평균 수치, 서비스 시간, 타임 스탬프, 배치 작업, 캐시 만료

### 단조 시계 대 일 기준 시계
- time-of-day clock
- monotonic clock

#### 일 기준 시계
- 동기화가 필요하다.
- 일반적으로 NTP(Network Time Protocol)을 통해서 동기화 된다.
- 해상도가 낮다.

#### 단조 시계
- cpu 마다 있을 수 있다.
- 일반적으로 단일 머신 내부의 시간을 측정하는데에 적합하다. (Stop-the-world 미고려)

#### 시계 동기화와 정확도
- drift 현상(드리프트 현상) : 시계에 오차가 생긴다. 대부분 재동기화를 통해서 해결한다.
- 동기화 시도시 너무 많은 차이가 존재하면 동기화가 거부될 수 있다.
- 방화벽에 동기화가 막힐 수도 있다.
- 아무리 해봐야 네트워크 지연은 극복 못한다.
- 모든 NTP 서버를 신뢰할 수 없다.
- 윤초는 언제나 문제다. - 대부분 이를 해결하려고 리눅스 타임을 쓴다.

#### 동기화된 시계에 의존하기
- 시계가 잘못되었는지 알 방법이 없다.
- 하지만 이게 제일 속편한 방법이다. -> 보통은 시계를 잘- 동기화 시켜놓고 사용한다.

#### 이벤트 순서화용 타임 스템프
- LWW(last write wins):
  - 데이터베이스 쓰기가 불가사의하게 사라질 수 있다.
  - 인과성 추적 메커니즘이 필요하다.
  - 해상도가 낮다면 동일한 타임 스탬프가 존재한다.
- Clock Skew 문제

#### 시계 읽기는 신뢰 구간이 있다.
- Spanner, Google TrueTime API

#### 전역 스냅숏용 동기화된 시계
- 가장 흔한 스냅숏 격리 구현 : 단조 증가하는 트랜잭션 ID
  - 하지만 분산 디비라면?
- 동기화된 타임스탬프를 이용해서 해결하려고 한다.
- 아직까지는 연구 단계다.

### 프로세스 중단
- 임차권(lease) : 특정 시점에 오직 하나의 리더만이 임차권을 획득 가능하다.
  - 리더가 언제까지 확실한 리더인지 알 수 있다.

- 쓰레드가 오랫동안 멈추는 경우:
  - gc "stop-the-world"
  - vm 위에서 돌아가서 cpu를 많이 못받게 되면
  - 하드웨어적 일시 정지
  - steal time
  - waiting io
  - swap (threshing)
  - SIGSTOP

### 응답 시간 보장
- deadline, hard real-time
- RTOS
- 참고:
  - [linux scheduler 클래스](https://makerdark98.dev/wiki/linux-debug/scheduling.html#%EC%8A%A4%EC%BC%80%EC%A4%84%EB%9F%AC-%ED%81%B4%EB%9E%98%EC%8A%A4)

### 가비지 컬렉션의 영향을 제한하기
- gc 중단을 예고하고 실행하는 동안 다른 노드가 우선권을 가지게


---
- 정말로 엔지니어들은 시계를 신경 쓸까?
  - https://stackoverflow.com/questions/48949090/what-time-is-it-in-a-kubernetes-pod
  - https://www.goglides.dev/bkpandey/ntp-in-a-kubernetes-cluster-1l0a
  - [Clock Skew report](https://pingcap.com/blog/simulating-clock-skew-in-k8s-without-affecting-other-containers-on-node)

  - 결론 : 대부분의 경우 NTP 를 신뢰한다.
  - 이유에 대한 개인 생각 :
    - TrueTime 같은 경우 아직 비용이 크고, 모든 개발자들에게 익숙하지 않다.
    - 신뢰구간을 얻는 것이 실제로 문제를 해결해주지 않는다. 오히려 문제를 더 복잡하게 만들 수 있다.
    - 대부분의 경우 node 들은 지역성이 존재한다.

- 정말로 엔지니어들은 네트워크 레이어를 잘 알아야할까? 또는 네트워크 레이어를 신뢰하면 안될까?
  - [아마존 AWS, 6일 밤 또 장애…게임·배달 서비스 등 마비](https://www.hani.co.kr/arti/economy/it/1030006.html)

- 데이터센터는 우리가 익숙한 네트워크 레이어를 사용할까?

- 그렇다면 어떻게 테스트하는가?
  - [litmuschaos - docs](https://litmuschaos.github.io/litmus/experiments/categories/pods/pod-network-partition/)
