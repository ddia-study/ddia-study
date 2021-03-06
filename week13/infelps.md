# 스트림 처리

- 일간 일괄 처리의 문제점은 입력의 변화가 하루가 지나야 반영된다는 것이다.
- 고정된 시간 조각이라는 개념을 완전히 버리고 단순히 이벤트가 발생할 때마다 처리하는 것이 스트림 처리의 기본 개념이다.

## 이벤트 스트림 전송

- 입력이 파일일 때 대개 첫 번째 단계로 파일을 분석해 레코드의 연속으로 바꾸는 처리를 한다. 스트림 처리 문맥에서 레코드는 보통 이벤트라고 한다.
- 이벤트는 일반적으로 일기준 시계를 따르는 이벤트 발생 타임스탬프를 포함한다.
- 생산자가 이벤트를 한 번 만들면 해당 이벤트를 복수의 소비자, 구독자, 수신자가 처리할 수 있다.
- 폴링이 잦을수록 폴링을 수행하는 오버헤드가 커진다.
- 새로운 이벤트가 나타날 때마다 소비자에게 알리는 편이 낫다.

### 메시징 시스템

- 새로운 이벤트에 대해 소비자에게 알려주려고 쓰이는 일반적인 방법은 메시징 시스템을 사용하는 것이다.
- 가장 간단한 방법은 유닉스 파이프나 TCP 연결과 같은 직접 통신 채널을 사용하는 방법이다. 그러나 메시징 시스템 대부분은 이 기본 모델을 확장한다.
- 발행/구독 모델에서는 여러 시스템들이 다양한 접근법을 사용한다.
- 두 가지 질문(시스템을 구별하는 데 상당히 도움이 된다)
	- 생산자가 소비자가 메시지를 처리하는 속도보다 빠르게 메시지를 전송한다면 어떻게 될까? : 메시지를 버리거나 큐에 버퍼링하거나 생산자가 메시지를 더 보내지 못하게 막는다.
	- 노드가 죽거나 일시적으로 오프라인이 된다면 어떻게 될까? 손실되는 메시지가 있을까? : 지속성을 갖추려면 디스크에 기록하거나 복제본 생성을 하거나 둘 모두를 해야 한다.

### 생산자에서 소비자로 메시지를 직접 전달하기

- 직접 메시징 시스템은 설계 상황에서는 잘 동작하지만 일반적으로 메시지가 유실될 수 있는 가능성을 고려해서 애플리케이션 코드를 작성해야 한다.
- 소비자가 오프라인이라면 메시지를 전달하지 못하는 상태에 있는 동안 전송된 메시지는 잃어버릴 수 있다.

### 메시지 브로커

- 메시지 브로커는 근본적으로 메시지 스트림을 처리하는 데 최적화된 데이터베이스의 일종이다.
- 생산자는 브로커로 메시지를 전송하고 소비자는 브로커에서 메시지를 읽어 전송받는다.
- 브로커에 데이터가 모이기 때문에 이 시스템은 클라이언트의 상태 변경에 쉽게 대처할 수 있다.
- 큐 대기를 하면 소비자는 일반적으로 비동기로 동작한다.

### 메시지 브로커와 데이터베이스의 비교

- 데이터베이스는 명시적으로 데이터가 삭제될 떄까지 데이터를 보관한다. 반면 메시지 브로커 대부분은 소비자에게 데이터 배달이 성공할 경우 자동으로 메시지를 삭제한다.
- 메시지 브로커는 대부분 메시지를 빨리 지우기 때문에 작업 집합이 상당히 작다고 가정한다. 즉 큐 크기가 작다.
- 데이터베이스는 보조 색인을 지원하고 데이터 검색을 위한 다양한 방법을 지원하는 반면 메시지 브로커는 특정 패턴과 부합하는 토픽의 부분 집합을 구독하는 방법을 지원한다.
- 데이터베이스에 질의할 때 그 결과는 일반적으로 질의 시점의 데이터 스냅숏을 기준으로 한다. 반대로 메시지 브로커는 임의 질의를 지원하지 않지만 데이터가 변하면 클라이언트에게 알려준다.

### 복수 소비자

- 로드 밸런싱 : 각 메시지는 소비자 중 하나로 전달된다. 따라서 소비자들은 해당 토픽의 메시지를 처리하는 작업을 공유한다. 브로커는 메시지를 전달할 소비자를 임의로 지정한다. JMS(공유 구독)
- 팬 아웃 : 각 메시지는 모든 소비자에게 전달된다. 팬 아웃 방식을 사용하면 여러 독립적인 소비자가 브로드캐스팅된 동일한 메시지를 서로 간섭 없이 "청취"할 수 있다.

### 확인 응답과 재전송

- 메시지를 잃어버리지 않기 위해 메시지 브로커는 확인 응답을 사용한다. 클라이언트는 메시지 처리가 끝났을 때 브로커가 메시지를 큐에서 제거할 수 있게 브로커에게 명시적으로 알려야 한다.
- 브로커가 확인 응답을 받기 전에 클라이언트로의 연결이 닫히거나 타임아웃되면 브로커는 메시지가 처리되지 않았다고 가정하고 다른 소비자에게 다시 전송한다.
- 부하 균형 분산과 결합할 떄 이런 재전송 행위는 메시지 순서에 영향을 미친다.
- 소비자마다 독립된 큐를 사용하면, 즉 부하 균형 분산 기능을 사용하지 않는다면 이 문제를 피할 수 있다.

### 파티셔닝된 로그

- 브로커가 확인 응답을 받으면 브로커에서 메시지를 삭제하기 때문에 이미 받은 메시지는 복구할 수 없다.
- 소비자를 다시 실행해도 동일한 결과를 받지 못한다.
- 테이터베이스의 지속성 있는 저장 방법과 메시징 시스템의 지연 시가닝 짧은 알림 기능을 조합하는 것이 로그 기반 메시지 브로커의 기본 아이디어다.

### 로그를 사용한 메시지 저장소

- 생산자가 보낸 메시지는 로그 끝에 추가하고 소비자는 로그를 순차적으로 읽어 메시지를 받는다.
- 소비자가 로그 끝에 도달하면 새 메시지가 추가됐다는 알림을 기다린다.
- 디스크 하나를 쓸 때보다 처리량을 높이기 위해 확장하는 방법으로 로그를 파티셔닝하는 방법이 있다.
- 각 파티션 내에서 브로커는 모든 메시지에 오프셋이라고 부르는, 단조 증가하는 순번을 부여한다. 다른 파티션 간 메시지의 순서는 보장하지 않는다

### 로그 방식과 전통적인 메시징 방식의 비교

- 로그 기반 접근법은 당연히 팬 아웃 메시징 방식을 제공한다. 소비자가 서로 영향 없이 독립적으로 로그를 읽을 수 있고 메시지를 읽어도 로그에서 삭제되지 않기 때문이다.
- 개별 메시지를 소비자 클라이언트에게 할당하지 않고 소비자 그룹 간 로드 밸런싱하기 위해 브로커는 소비자 그룹의 노드들에게 전체 파티션을 할당할 수 있다.
- 소비자에 로그 파티션이 할당되면 소비자는 단일 스레드로 파티션에서 순차적으로 메시지를 읽는다.
	- 토픽 하나를 소비하는 작업을 공유하는 노드 수는 많아야 해당 토픽의 로그 파티션 수로 제한된다.
	- 특정 메시지 처리가 느리면 파티션 내 후속 메시지 처리가 지연된다.

### 소비자 오프셋

- 주기적으로 소비자 오프셋을 기록하면 추적 오버헤드가 감소하고 일괄 처리와 파이프라이닝을 수행할 수 있는 기회를 제공해 로그 기반 시스템의 처리량을 늘리는 데 도움을 준다.
- 메시지 오프셋은 단일 리더 데이터베이스 복제에서 널리 쓰는 로그 순차 번호와 상당히 유사하다.
- 소비자 노드에 장애가 발생하면 소비자 그룹 내 다른 노드에 장애가 발생한 소비자의 파티션을 할당하고 마지막 기록된 오프셋부터 메시지를 처리하기 시작한다.

### 디스크 공간 사용

- 디스크 공간을 재사용하기 위해 로그를 여러 조각으로 나누고 가끔 오래된 조각을 삭제하거나 보관 저장소로 이동한다.
- 소비자 처리 속도가 느려 메시지가 생산되는 속도를 따라잡지 못하면 소비자가 너무 뒤처져 소비자 오프셋이 이미 삭제한 조각을 가리킬 수도 있다.
- 원형 버퍼, 링 버퍼 : 로그는 크기가 제한된 버퍼로 구현하고 버퍼가 가득 차면 오래된 메시지 순서대로 버린다.
- 이 시스템은 큐가 작을 때는 빠르지만 디스크에 기록하기 시작하면 매우 느려진다.


### 소비자가 생산자를 따라갈 수 없을 때

- 브로커는 버퍼 크기를 넘는 오래된 메시지를 자연스럽게 버린다.
- 소비자가 로그의 헤드로부터 얼마나 떨어졌는지 모니터링하면 눈에 띄게 뒤처지는 경우 경고할 수 있다.
- 어떤 소비자가 너무 뒤처져서 메시지를 잃기 시작해도 해당 소비자만 영향을 받고 다른 소비자들의 서비스를 망치지는 않는다.


### 오래된 메시지 재생

- 로그 기반 메시지 브로커는 메시지를 소비하는 게 파일을 읽는 작업과 유사한데 로그를 변화시키지 않는 읽기 전용 연산이기 때문이다.
- 소비자의 출력을 제외한, 메시지 처리의 유일한 부수 효과는 소비자 오프셋 이동이다.
- 하지만 소비자 오프셋은 소비자 관리 아래에 있다. 그래서 필요하다면 쉽게 조작할 수 있다.
- 로그 기반 메시징 시스템은 많은 실험을 할 수 있고 오류와 버그를 복구하기 쉽기 때문에 조직 내에서 데이터플로를 통합하는 데 좋은 도구다. 

## 데이터베이스와 스트림

- 메시징과 스트림에서 아이디어를 가져와 데이터베이스에 적용할 수 있다.

### 시스템 동기화 유지하기

- 데이터베이스에 아이템 하나를 갱신하면 캐시와 색인과 데이터 웨어하우스도 마찬가지로 갱신해야 한다.
- 데이터베이스 전체를 복사하고 변환한 후 데이터 웨어하우스로 벌크 로드한다. 이 과정은 일괄 처리다.
- 주기적으로 데이터베이스 전체를 덤프하는 작업이 너무 느리면 대안으로 사용하는 방법으로 이중 기록이 있다.
	- 데이터가 변할 때마다 애플리케이션 코드에서 명시적으로 각 시스템에 기록한다.
	- 경쟁 조건
	- 한쪽 쓰기가 성공할 때 다른 쪽 쓰기는 실패할 수 있다. 내결함성 문제(두 시스템 간 불일치가 발생)

### 변경 데이터 캡처

- 변경 데이터 캡처는 데이터베이스에 기록하는 모든 데이터의 변화를 관찰해 다른 시스템으로 데이터를 복제할 수 있는 형태로 추출하는 과정이다.
- 데이터베이스의 변경 사항을 캡처해 같은 변경 사항을 검색 색인에 꾸준히 반영할 수 있다.

### 변경 데이터 캡처의 구현

- 검색 색인과 데이터웨어하우스에 저장된 데이터는 레코드 시스템에 저장된 데이터의 또 다른 뷰일 뿐이므로 로그 소비자를 파생 데이터 시스템이라 할 수 있다.
- 변경 데이터 캡처는 본질적으로 변경 사항을 캡처할 데이터베이스 하나를 리더로 하고 나머지를 팔로워로 한다. 
- 로그 기반 메시지 브로커는 원본 데이터베이스에서 변경 이벤트를 전송하기에 적합하다.
- 데이터베이스 트리거를 사용하기도 한다.
	- 이 방식은 고장 나기 쉽고 성능 오버헤드가 상당하다.

- 변경 데이터 캡처는 메시지 브로커와 동일하게 비동기 방식으로 동작한다.
	- 느린 소비자가 추가되더라도 레코드 시스템에 미치는 영향이 적다.
	- 복제 지연의 모든 문제가 발생하는 단점이 있다.

### 초기 스냅숏

- 대부분 모든 변경 사항을 영구적으로 보관하는 일은 디스크 공간이 너무 많이 필요하고 모든 로그를 재생하는 작업도 너무 오래 걸린다. 그래서 로그를 적당히 잘라야 한다.
- 최근에 갱신하지 않은 항목은 로그에 없기 때문에 최근 사항만 반영하는 것으로는 충분하지 않다. 따라서 일관성 있는 스냅숏을 사용해야 한다.
- 데이터베이스 스냅숏은 변경 로그의 위치나 오프셋에 대응돼야 한다. 그래야 스냅숏 이후에 변경 사항을 적용할 시점을 알 수 있다.

### 로그 컴팩션

- 저장 엔진은 주기적으로 같은 키의 로그 레코드를 찾아 중복을 제거하고 각 키에 대해 가장 최근에 갱신된 내용만 유지한다.
- 파생 데이터 시스템을 재구축할 때마다 새 소비자는 컴팩션된 로그 토픽의 오프셋 0부터 시작해서 순차적으로 데이터베이스의 모든 키를 스캔하면 된다.

### 이벤트 소싱

- 이벤트 소싱은 변경 데이터 캡처와 유사하게 애플리케이션 상태 변화를 모두 변경 이벤트 로그로 저장한다.
- 애플리케이션 관점에서 사용자의 행동을 불변 이벤트로 기록하는 방식은 변경 가능한 데이터베이스 상에서 사용자의 행동에 따른 효과를 기록하는 방식보다 훨씬 유의미하다.

### 이벤트 로그에서 현재 상태 파생하기

- 시스템에 기록한 데이터를 표현한 이벤트 로그를 가져와 사용자에게 보여주기에 적당한 애플리케이션 상태로 변환해야 한다.
- 로직을 자유롭게 사용할 수 있지만 결정적 과정이어야 한다.

### 명령과 이벤트

- 이벤트 소싱 철학은 이벤트와 명령을 구분하는 데 주의한다.
- 사용자 요청이 처음 도착했을 때 이 요청은 명령이다.
- 무결성이 검증되고 명령이 승인되면 명령은 지속성 있는 불변 이벤트가 된다.
- 이벤트는 생성 시점에 사실이 된다.
- 이벤트 스트림 소비자는 이벤트를 거절하지 못한다. 소비자가 이벤트를 받은 시점에는 이벤트는 이미 불변 로그의 일부분이다.

### 상태와 스트림 그리고 불변성

- 상태가 변할 때마다 해당 상태는 시간이 흐름에 따라 변한 이벤트의 마지막 결과다.
- 상태가 어떻게 바뀌었든 항상 이런 변화를 일으킨 일련의 이벤트가 있다.
- 애플리케이션 상태를 시간에 따른 이벤트 스트림을 적분해서 구할 수 있고 변경 스트림은 시간으로 상태를 미분해서 구할 수 있다.

### 불변 이벤트의 장점

- 우연히 버그가 있는 코드를 배포해서 데이터베이스에 잘못된 데이터를 기록했을 때 코드가 데이터를 덮어썻다면 복구하기가 매우 어렵다. 
- 추가만 하는 불변 이벤트 로그를 썼다면 문제 상황의 진단과 복구가 훨씬 쉽다.

### 동일한 이벤트 로그로 여러 가지 뷰 만들기

- 불변 이벤트 로그에서 가변 상태를 분리하면 동일한 이벤트 로그로 다른 여러 읽기 전용 뷰를 만들 수 있다.
- 이벤트 로그에서 데이터베이스로 변환하는 명시적인 단계가 있으면 시간이 흐름에 따라 애플리케이션을 발전시키기 쉽다.
- 명령과 질의 책임의 분리 : 데이터를 쓰는 형식과 읽는 형식을 분리해 다양한 읽기 뷰를 허용한다면 상당한 유연성을 얻을 수 있다.
- 읽기 최적화된 뷰는 데이터를 비정규화하는 것이 전적으로 합리적이다.

### 동시성 제어

- 이벤트 소싱과 변경 데이터 캡처의 가장 큰 단점은 이벤트 로그의 소비가 대개 비동기로 이뤄진다는 점이다.
- 사용자가 로그에 이벤트를 기록하고 이어서 로그에서 파생된 뷰를 읽어도 기록한 이벤트가 아직 읽기 뷰에 반영되지 않았을 가능성이 있다.
- 읽기 뷰의 갱신과 로그에 이벤트를 추가하는 작업을 동기식으로 수행하는 방법
	- 트랜잭션에서 여러 쓰기를 원자적 단위로 결합해야 하므로 이벤트 로그와 읽기 뷰를 같은 저장 시스템에 담아야 한다.
	
- 이벤트 로그로 현재 상태를 만들면 동시성 제어 측면이 단순해진다.
	- 이벤트 소싱을 사용하면 사용자 동작에 대한 설명을 자체적으로 포함하는 이벤트를 설계할 수 있다.
	- 사용자 동작은 한 장소에서 한 번 쓰기만 필요하다.

- 이벤트 로그와 애플리케이션 상태를 같은 방식으로 파티셔닝하면 간단한 단일 스레드 로그 소비자는 쓰기용 동시성 제어는 필요하지 않다.

### 불변성의 한계

- 상대적으로 작은 데이터셋에서 매우 빈번히 갱신과 삭제를 하는 작업부하는 불변 히스토리가 감당하기 힘들 정도로 커지거나 파편화 문제가 발생할 수 있다.
- 데이터가 모두 불변성임에도 관리상의 이유로 데이터를 삭제할 필요가 있는 상황일 수 있다.
- 데이터를 진짜로 삭제하는 작업은 놀라울 정도로 어렵다. 많은 곳에 복제본이 남아 있기 때문이다.

## 스트림 처리

- 스트림을 처리하는 방법
	-	이벤트에서 데이터를 꺼내 데이터베이스나 캐시, 검색 색인 또는 유사한 저장소 시스템에 기록하고 다른 클라이언트가 이 시스템에 해당 데이터를 질의한다.
	-	이벤트를 사용자에 직접 보낸다.
	-	하나 이상의 입력 스트림을 처리해 하나 이상의 출력 스트림을 생산한다.

- 스트림을 처리하는 코드 조각을 연산자나 작업이라 부른다.
- 일괄 처리 작업과 가장 크게 다른 점은 스트림은 끝나지 않는다는 점이다.


### 복잡한 이벤트 처리

- 특정 이벤트 패턴을 검색해야 하는 애플리케이션에 특히 적합하다.
-  감지할 이벤트 패턴을 설명하는 데 종종 SQL 같은 고수준 선언형 질의 언어나 그래픽 사용자 인터페이스를 사용하기도 한다.
-  질의는 오랜 기간 저장되고 입력 스트림으로부터 들어오는 이벤트는 지속적으로 질의를 지나 흘러가면서 이벤트 패턴에 매칭되는 질의를 찾는다.

### 스트림 분석

- 분석은 연속한 특정 이벤트 패턴을 찾는 것보다 대량의 이벤트를 집계하고 통계적 지표를 뽑는 것을 더 우선한다.
- 일반적으로 이런 통계는 고정된 시간 간격 기준으로 계산한다. 이때 집계 시간 간격을 윈도우라 한다.
- 스트림 분석 시스템은 확률적 알고리즘을 사용하기도 한다.


### 구체화 뷰 유지하기

- 어떤 데이터셋에 대한 또 다른 뷰를 만들어 효율적으로 질의할 수 있게 하고 기반이 되는 데이터가 변경될 때마다 뷰를 갱신한다.
- 이벤트 소싱에서 애플리케이션 상태는 이벤트 로그를 적용함으로써 유지된다.
- 구체화 뷰를 만들려면 잠재적으로 임의의 시간 범위에 발생한 모든 이벤트가 필요하다.

### 스트림 상에서 검색하기

- 전문 검색 질의와 같은 복잡한 기준을 기반으로 개별 이벤트를 검색해야 하는 경우도 있다.
- 예를 들어 미디어 모니터링 서비스는 미디어 아웃렛에서 새 기사와 방송 피드를 구독하고 관심 있는 회사나 상품 또는 주제를 언급하는 뉴스를 검색한다.
- 사전에 검색 질의를 설정하면 꾸준히 스트림으로 들어오는 뉴스 항목을 이 질의에 매칭한다.

### 메시지 전달과 RPC

- 유사 RPC 시스템과 스트림 처리 사이에 겹치는 영역이 있다.
- 아파치 스톰에는 분산 RPC라 부르는 기능이 있다.
- 이벤트 스스림을 처리하는 노드 집합에 질의를 맡길 수 있다.
- 이 질의는 입력 스트림 이벤트가 끼워지고 그 결과들을 취합해 사용자에게 돌려준다.

### 시간에 관한 추론

- 스트림 처리자는 종종 시간을 다뤄야 할 때가 있다.
- 이때 주로 "지난 5분 동안 평균"같은 시간 윈도우를 자주 사용한다.
- 많은 스트림 처리 프레임워크는 윈도우 시간을 결정할 때 처리하는 장비의 시스템 시계를 이용한다.
- 눈에 띌 정도로 처리가 지연되면, 즉 이벤트가 실제로 발생한 시간보다 처리 시간이 많이 늦어지면 문제가 생긴다.

### 이벤트 시간 대 처리 시간

- 메시지가 지연되면 메시지 순서를 예측하지 못할 수도 있다.
- 이벤트 시간과 처리 시간을 혼동하면 좋지 않은 데이터가 만들어진다.
- 그림 11-7 처리 시간 기준으로 윈도우를 만들면 처리율의 변동 때문에 생기는 허상을 남긴다.

### 준비 여부 인식

- 이벤트 시간 기준으로 윈도우를 정의할 때 발생하는 까다로운 문제는 특정 윈도우에서 모든 이벤트가 도착했다거나 아직도 이벤트가 계속 들어오고 있는지를 확신할 수 없다는 점이다.
- 윈도우를 이미 종료한 후에 도착한 낙오자 이벤트를 처리할 방법이 필요하다.
	-	낙오자 이벤트는 무시한다.
	-	수정 값을 발행한다.

### 어쨌든 어떤 시계를 사용할 것인가?

- 이벤트가 시스템의 여러 지점에 버퍼링됐을 때 이벤트에 타임스탬프를 할당하는 것은 더 어렵다.
- 우연히 또는 고의로 잘못된 시간이 설정됐을 가능성이 있기 떼문에 사용자가 제어하는 장비의 시계를 항상 신뢰하기는 어렵다.
- 잘못된 장치 시계를 조정하는 한 가지 방법은 세 가지 타임스탬프를 로그로 남기는 것이다.
	- 이벤트가 발생한 시간
	- 이벤트를 서버로 보낸 시간
	- 서버에서 이벤트를 받은 시간

- 두 번째와 세 번째의 타임스탬프 차이를 구하면 장치 시계와 서버 시계 간의 오프셋을 추정할 수 있다.

### 윈도우 유형

- 텀블링 윈도우 
- 홉핑 윈도우
- 슬라이딩 윈도우
- 세션 윈도우

### 스트림 조인

- 스트림 상에서 새로운 이벤트가 언제나 나타날 수 있다는 사실은 스트림 상에서 수행하는 조인을 일괄 처리 작업에서 수행하는 조인보다 훨씬 어렵게 만든다.

### 스트림 스트림 조인(윈도우 조인)

- 네트워크 지연도 가변적이기 때문에 클릭 이벤트가 검색 이벤트보다 먼저 도착할 수 있다.
- 조인을 위한 적절한 윈도우 선택이 필요하다.
- 이런 유형의 조인을 구현하려면 스트림 처리자가 상태를 유지해야 한다.

### 스트림 테이블 조인(스트림 강화)

- 사용자 활동 이벤트 집합과 사용자 프로필 데이터베이스를 조인하는 예제
- 입력은 활동 이벤트 스트림이고 출력은 사용자 프로필 정보가 추가된 활동 이벤트
- 이 과정을 데이터베이스의 정보로 활동 이벤트를 강화한다고 한다.