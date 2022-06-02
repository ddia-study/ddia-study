# 분산 시스템의 골칫거리

## 지식, 진실, 그리고 거짓말

- 네트워크에 있는 노드는 어떤 것도 확실히 알지 못한다.
- 네트워크를 통해 받은 메시지를 기반으로 추측만 할 수 있을 뿐이다.
- 분산 시스템에서 우리는 동작(시스템 모델)에 관해 정한 가정을 명시하고, 이런 가정을 만족시키는 방식으로 실제 시스템을 설계 할 수 있다.

### 진실은 다수결로 결정된다

- 노드가 자신에게 보내지는 메시지는 모두 받을 수 있지만 그 노드에서 밖으로 나가는 메시지는 유실되거나 지연되는 경우
  - 타임아웃이 난 후 다른 노드들은 그 노드를 죽었다고 선언

- 한쪽 연결이 끊긴 노드가 자신이 보내는 메시지가 다른 노드로부터 확인 응답을 받지 못하는 것을 알아내서 네트워크에 결함이 있는 걸 깨달는 경우
  - 다른 노드들이 그 노드가 죽었다고 선언, 한쪽 연결이 끊긴 노드는 아무 일도 할 수 없음

- 긴 stop-the-world 가비지 컬렉션 중단을 경함하는 노드
  - 다른 노드들은 기다린 후 재시도하다 결국엔 노드가 죽었다고 선언
  - GC가 끝나고 노드의 스레드들은 실행을 재개
  - GC가 실행되는 노드는 자신이 죽은것으로 선언됐다는 것조차 알지 못함

- 분산 시스템은 한 노드에만 의존할 수 없다.
- 여러 분산 알고리즘은 정족수, 노드들 사이의 투표에 의존한다.
- 노드의 과반수 이상을 정족수로 삼는 게 가장 흔하다.(합의 알고리즘)
  
### 리더와 잠금

- 시스템이 오직 하나의 뭔가가 필요한 때
  - 스플릿 브레인을 피하기 위해 오직 한 노드만 데이터베이스 파티션의 리더가 될 수 있다.
  - 특정한 자원이나 객체에 동시에 쓰거나 오염시키는 것을 방지하기 위해 오직 하나의 트랜잭션이나 클라이언트만 어떤 자원이나 객체의 잠금을 획득할 수 있다.
  - 사용자명으로 사용자를 유일하게 식별할 수 있어야 하므로 오직 한 명의 사용자만 특정한 사용자명으로 등록할 수 있다.

-  어떤 노드가 스스로를 "선택된 자"라고 믿을지라도 노드의 정족수도 반드시 동의한다는 뜻이 아니다.
   - 이전에 리더였지만 시간이 흐른 후 다른 노드들이 죽었다고 판단하여 강등되고 다른 리더가 선출됐을지도 모른다.
   - 노드가 선택된 자인 것처럼 계속 행동한다면 신중하게 설계되지 않은 시스템에서는 문제를 유발할 수 있다.
   - 그림 8-4(잠금을 잘못 구현해서 생긴 데이터 오염 버그)

### 펜싱 토큰

- 리소스에 대한 접근을 보호하기 위해 잠금이나 임차권을 쓸 떄, 자신이 "선택된 자"라고 잘못 믿고 있는 노드가 나머지 시스템을 방해할 수 없도록 보장해야 한다.
- 펜싱(fencing)
  - 그림 8-5
  - 잠금 서버가 잠금이나 임차권을 승인할 때마다 펜싱 토큰도 반환한다고 가정한다.
  - 펜싱 토큰은 잠금이 승인될 때마다 증가하는 숫자
  - 클라이언트가 쓰기 요청을 저장소 서비스로 보낼 때마다 자신의 현재 펜싱 토큰을 포함
  - 주키퍼 : 트랜잭션 ID zxid, 노드 버전 cversion을 펜싱 토큰으로 사용(단조 증가 보장)
  - 스스로를 뜻하지 않게 폭력적인 클라이언트로부터 보호하려는 서비스는 서버 측에서 토큰을 확인하는 게 좋다.

### 비잔틴 결함

- 노드가 고의로 시스템의 보장을 무너뜨리려한다면 가짜 펜싱 토큰을 포함한 메시지를 보내기만 하면 된다.
- 비잔틴 결함 : 어떤 노드가 실제로는 받지 않은 특정 메시지를 받았다고 주장하는 동작, 비잔틴 장군 문제
- 비잔틴 내결함성 : 일부 노드가 오작동하고 프로토콜을 준수하지 않거나 악의적인 공격자가 네트워크를 방해하더라도 시스템이 계속 올바르게 동작함
  - 항공우주 산업 환경에서 컴퓨터의 메모리나 CPU 레지스터에 저장된 데이터는 방사선에 오염돼서 다른 노드에게 전혀 예측할 수 없는 방식으로 반응할 수 있음
  - 여러 조직이 참여하는 시스템에서 어떤 참여자들은 다른 사람을 속이거나 사취하려고 할지도 모른다. 비트코인

- 웹 애플리케이션은 최종 사용자가 제어하는 웹브라우저 같은 클라이언트의 행동이 임의적이고 악의적이라고 예상해야 한다.
  - 입력 확인, 살균, 출력 이스케이핑이 매우 중요한 이유
  - 크로스 사이트 스크립팅 : https://nordvpn.com/ko/blog/xss-attack/
  - SQL 주입 공격 : https://noirstar.tistory.com/264
  - 중앙 권한이 없는 피어투피어 네트워크에 더 적절하다.

- 동일한 소프트웨어를 모든 노드에 배포하면 비잔틴 내결함성 알고리즘도 도움이 되지 않는다.
- 전통적인 메커니즘이 여전히 공격자로부터 보호하는 수요 수단으로 사용되고 있다.

### 약한 형태의 거짓말

- 보호 메커니즘은 완강한 적에게는 대항할 수 없으므로 완전한 비잔틴 내결함성을 지니지는 않지만 그럼에도 더욱 나은 신뢰성으로 향하는 간단하고 실용적인 발걸음이다. 
  - 네트워크 패킷은 때때로 하드웨어 문제나 운영체제, 드라이버, 라우터 등의 버그 때문에 오염된다. 어플리케이션 수준 프로토콜에서 체크섬을 쓰면 충분하다.
  - 공개적으로 접근 가능한 애플리케이션은 사용자 입력을 신중하게 살균해야 한다. 
  - NTP 클라이언트는 여러 서버 주소를 설정할 수 있다. 동기화를 할 때 클라이언트는 모든 서버에 접속해서 그들의 오차를 추정한 후 서버 중 다수가 어떤 시간 범위에 동의하는지 확인할 수 있다.

### 시스템 모델과 현실

- 알고리즘은 실행되는 하드웨어와 소프트웨어 설정의 세부 사항에 너무 심하게 의존하지 않는 방식으로 작성해야 한다.
- 시스템 모델을 정의해서 정형화하는데, 시스템 모델은 알고리즘이 가정하는 것을 기술한 추상화다.
- 타이밍 가정
  - 동기식 모델 : 네트워크 지연, 프로세스 중단, 시계 오차에 모두 제한이 있다고 가정한다.
  - 부분 동기식 모델 : 시스템이 대부분의 시간에는 동기식 시스템처럼 동작하지만 때때로 네트워크 지연, 프로세스 중단, 시계 드리프트의 한계치를 초과한다.
  - 비동기식 모델 : 알고리즘은 타이밍에 대한 어떤 가정도 할 수 없다.

- 노드 장애
  - 죽으면 중단하는 결함 : 노드에 장애가 나는 방식이 하나뿐, 다시 말해 죽는 것뿐이라고 가정할 수 있다.
  - 죽으면 복구하는 결함 : 노드가 어느 순간에 죽을 수 있지만 알려지지 않은 시간이 흐른 후에는 아마도 다시 응답하기 시작할 것이라고 가정한다.
  - 비잔틴 결함 : 노드는 다른 노드를 속이거나 기만하는 것을 포함해 전적으로 무슨 일이든 할 수 있다.

### 알고리즘의 정확성

- 알고리즘이 정확하다는 게 어떤 의미인지 정의하기 위해 알고리즘의 속성을 기술할 수 있다.
- 펜싱 토큰 속성
  - 유일성 : 펜싱 토큰 요청이 같은 값을 반환하지 않는다.
  - 단조 일련번호 : 요청 x가 토큰 tx를, 요청 y가 토큰 ty를 반환했고 y가 시작하기 전에 x가 완료됐다면 tx&lt;ty를 만족한다.
  - 가용성 : 펜싱 토큰을 요청하고 죽지 않은 노드는 결국에는 응답을 받는다.

- 알고리즘은 시스템 모델에서 발생하리라고 가정한 모든 상황에서 그 속성들을 항상 만족시키면 해당 시스템 모델에서 정확하다.

### 안전성과 활동성

- 안전성 : 비공식적으로 나쁜 일은 일어나지 않는다
- 활동성 : 좋은 일은 결국 일어난다
- 안전성 속성이 위반되면 그 속성이 깨진 특정 시점을 가리킬 수 있다. 안전성 속성이 위반된 후에는 그 위반을 취소할 수 없다.
- 어떤 시점을 정하지 못할 수 있지만 항상 미래에 그 속성을 만족시킬 수 있다는 희망이 있다.
- 분산 알고리즘은 시스템 모델의 모든 상황에서 안전성 속성이 항상 만족되기를 요구하는 게 일반적이다.
  - 모든 노드가 죽거나 네트워크 전체에 장애가 생기거라도 알고리즘은 잘못된 결과를 반환하지 않는다고 보장해야 한다.

- 활동성 속성에 대해서는 경고를 하는게 허용된다.
  - 노드의 다수가 죽지 않고 네트워크가 중단으로부터 결국 복구됐을 때만 요청이 응답을 받아야 한다.
  - 부분 동기식 모델의 정의는 시스템이 결국 동기식 상태로 돌아오기를 요구한다.

### 시스템 모델을 현실 세계에 대응시키기

- 안전성 및 활동성 속성과 시스템 모델은 분산 시스템의 정확성을 따져보는데 매우 유용하다.
- 알고리즘을 이론적으로 설명할 때는 그냥 어떤 일이 일어나지 않는다고 가정할 수 있다.
- 실제 구현에는 여전히 불가능하다고 가정했던 일이 발생하는 경우를 처리하는 코드를 포함시켜야 할 수도 있다.
- 추상 시스템 모델은 현실 시스템의 복잡함에서 우리가 추론할 수 있는 관리 가능하 결함의 집합을 뽑아내서, 문제를 이해하고 체계적으로 해결하려고 노력할 수 있게 하는 데 엄청난 도움이 된다.
- 알고리즘이 올바르다고 증명됐더라도 반드시 현실 시스템에서의 구현도 언제나 올바르게 동작한다는 뜻은 아니다.