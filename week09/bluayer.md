## 지식, 진실, 그리고 거짓말

***분산 시스템에는 공유 메모리가 없고 지연 변동이 큰 신뢰할 수 없는 네트워크를 통해 메시지를 보낼 수 있을 뿐이며 부분 장애, 신뢰성 없는 시계, 프로세스 중단에 시달릴 수 있다.***

신뢰성 없는 시스템 모델에서 잘 동작하는 소프트웨어를 만드는 게 가능할지라도 그것이 간단하지는 않다.

### 진실은 다수결로 결정된다.

1. 노드가 자신에게 보내지는 메시지는 모두 받을 수 있지만 그 노드에서 밖으로 나가는 메시지는 유실되거나 지연된다. 즉, 노드는 죽었다고 판단된다.
2. 노드는 자신의 응답이 밖으로 나가지 못한다는 걸 알게 되었다. 그럼에도 죽었다고 판단된다.
3. stop-the-world를 경험하는 노드라고 하자. 아무 요청도 처리 못하고 응답도 하지 못한다. 노드는 정상적으로 돌아오지만, 이미 죽었다고 선언됐을 가능성이 있다.

분산 시스템은 한 노드에만 의존할 수는 없다.

대신 여러 분산 알고리즘은 **정족수(quorum)**, 즉 노드들 사이의 투표에 의존한다. 

노드의 과반수 이상을 정족수로 삼는게 가장 흔하다.

### 리더와 잠금

시스템이 오직 하나의 뭔가가 필요할 때가 자주 있다.

- 스플릿 브레인을 피하기 위해 오직 한 노드만 데이터베이스 파티션의 리더가 될 수 있다.
- 특정한 자원이나 객체에 동시 쓰기, 오염 방지를 위해 오직 하나의 트랜잭션이나 클라이언트만 잠금을 획득할 수 있다.
- 오직 한 명의 사용자만 특정한 사용자명으로 등록할 수 있다.

분산 시스템에서 이를 구현하려면 주의해야 한다.

**어떤 노드가 스스로를 선택된 자라고 믿을지라도 노드의 정족수도 반드시 동의한다는 뜻은 아니다!**

Hbase Example : 데이터 오염 사례

### 펜싱 토큰

잠금 서버가 잠금이나 임차권을 승인할 때마다 fencing token도 반환한다고 가정한다.

펜싱 토큰은 잠금이 승인될 때마다 증가하는 숫자다.

클라이언트가 쓰기 요청을 저장소 서비스로 보낼 때마다 자신의 현재 펜싱 토큰을 포함하도록 요구할 수 있다.

이 메커니즘은 자원 자체가 이미 처리된 것보다 오래된 토큰을 사용해서 쓰는 것을 거부함으로써 토큰을 확인하는 활동적인 역할을 맡아야 한다. 

서버 측에서 토큰을 확인하는 것은 결점으로 보이지만 거의 틀림없이 좋다.

**흥미로운 내용**

마틴 클레프만이 fencing token에 대해서 서술한 글에서는 Redis의 Redlock 알고리즘이 어떤 문제가 있는지 설명해놨다.

_Redis Redlock_ : DLM(Distributed Lock Manager)를 구현하는 알고리즘. N개의 마스터에서 Lock 이슈로 인해 일어날 수 있는 문제를 해결하기 위한 알고리즘이다. Quorum 이상의 인스턴스에서 Lock을 획득하면 lock을 얻은 것으로 보는 알고리즘.

> The algorithm relies on the assumption that while there is no synchronized clock across the processes, the local time in every process updates at approximately at the same rate, with a small margin of error compared to the auto-release time of the lock. 

그 와중에 시계 동기화가 문제 없다는 가정 하에 잘 작동한다고 한다.

- 클라이언트 경쟁 이슈로 인해 오염이 가능하기 때문에 펜싱 토큰을 쓰는 것이 필요하지만, Redlock에는 해당 메커니즘이 존재하지 않는다.
- Redis는 분산 노드를 사용할텐데, 단순한 카운터 같은 걸로 펜싱 토큰을 발급하기 쉽지 않다. 펜싱 토큰 발행에도 합의 알고리즘이 필요하다.

Redis Redlock : https://redis.io/docs/reference/patterns/distributed-locks/

저자의 글 : https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html

~~대충 다 읽어봤는데, 머리가 정말 아프다...~~

### 비잔틴 결함

분산 시스템의 문제는 노드가 "거짓말" (임의의 결함이 있거나 오염된 응답을 보냄)을 할지도 모른다는 위험이 있다면 훨씬 더 어려워진다. 

예를 들어 어떤 노드가 실제로는 받지 않은 특정 메시지를 받았다고 주장할 수도 있다.

이런 동작을 **Byzantine fault**라고 하며 이렇게 신뢰할 수 없는 환경에서 합의에 도달하는 문제를 **Byzantine Generals Problem**라고 한다.

- 항공우주 산업 환경에서 컴퓨터의 메모리나 CPU 레지스터에 저장된 데이터는 방사선에 오염돼서 그 컴퓨터가 다른 노드에게 전혀 예측할 수 없는 방식으로 반응할 수 있다.
- 여러 조직이 참여하는 시스템에서 어떤 참여자들을 속이거나 사취하려고 할지도 모른다. 

시스템이 Byzantine fault-tolerant하게 만드는 프로토콜은 매우 복잡하고 내결함성을 지닌 임베디드 시스템은 하드웨어 수준의 지원에 의존한다. 

관련해서 볼만한 자료 : https://medium.com/loom-network-korean/%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%9B%90%EB%A6%AC-%EC%9D%B4%ED%95%B4-%EC%A0%9C-1%EB%B6%80-%EB%B9%84%EC%9E%94%ED%8B%B4-%EA%B2%B0%ED%95%A8-%EB%B0%A9%EC%A7%80-89ad2b7d1b65

## 약한 형태의 거짓말

약한 형태의 거짓말로부터 보호해주는 메커니즘은 완전한 비잔팀 내결함성을 지니지는 않지만 그럼에도 더욱 나은 신뢰성으로 향하는 간단하고 실용적인 발걸음이다.

- 네트워크 패킷의 오염 검출을 피하는 경우
> It turns out that ZooKeeper was not catching unhandled exceptions from its critical threads, meaning that if one died, the ZooKeeper process would continue to run without it. - https://www.pagerduty.com/blog/the-discovery-of-apache-zookeepers-poison-packet/
- 사용자 입력 sanitization
- NTP 클라이언트 사용 시 여러 서버 쓰기

### 시스템 모델과 현실

시스템에서 발생할 것으로 예상되는 결함의 종류를 정형화 해야 하는데, 이를 위해 **시스템 모델**(알고리즘이 가정하는 것을 기술한 추상화)을 정의한다.

- 타이밍 가정
  - 동기식 모델 : 모든 것(네트워크 지연, 프로세스 중단, 시계 오차)에 제한이 있다는 모델.
  - 부분동기식 모델 : 대부분의 시간에는 동기식 시스템처럼 동작하지만 때떄로 한계치를 초과한다는 모델.
  - 비동기식 모델 : 타이밍에 대한 어떤 가정도 할 수 없으며, 심지어는 시계가 없을 수도 있다.
- 노드 장애
  - crash-stop 결함
  - crash-recovery 결함
  - 비잔틴 결함

현실 시스템을 모델링하는 데는 죽으면 복구하는 결함을 지닌 부분 동기식 모델이 일반적으로 가장 유용하다.

### 알고리즘의 정확성

알고리즘은 시스템 모델에서 발생하리라고 가정한 모든 상황에서 그 속성들을 항상 만족시키면 해당 시스템 모델에서 정확하다.

하지만 모든 노드가 죽거나 모든 네트워크 지연이 무한히 길어진다면 어떤 알고리즘도 아무것도 할 수 없다.

### safety and liveness

유일성, 단조 일련번호는 안전성 속성이지만 가용성은 활동성 속성이다. 

- 안전성 : 나쁜 일은 일어나지 않는다. 속성이 깨진 특정 시점을 가리킬 수 있고, 위반을 취소할 수 없다.
- 활동성 : 좋은 일은 eventually 일어난다. 어떤 시점을 정하지 못할 수 있지만 항상 미래에 그 속성을 만족시킬 수 있다는 희망이 있다.

분산 알고리즘은 시스템 모델의 모든 상황에서 **안전성 속성이 항상** 만족되기를 요구하는 게 일반적이다. 
