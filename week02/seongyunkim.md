# 02. 데이터 모델과 질의 언어

데이터 모델: 문제를 어떻게 생각해야 하는지, 소프트웨어가 할 수 있는 일과 없는 일에 큰 영향

- 데이터 모델의 계층 ⇒ 하위 계층의 복잡성을 숨김
    - 애플리케이션 개발자: 현실을 보고 객체나 데이터 구조, API 모델링
    - 데이터 구조 저장: JSON, XML, RDB 테이블 등 범용 데이터 모델
    - DB 개발자: 범용 데이터 모델을 메모리나 디스크 상에 바이트 단위 표현, 데이터 질의, 탐색, 조작, 처리할 수 있도록
    - 하드웨어 엔지니어: 전류, 파동 관점에서 바이트를 표현

목표: 범용 데이터 모델(관계형 vs 문서 vs 그래프) + 다양한 질의 언어 및 사례

## 관계형 모델과 문서 모델

관계형 모델: 데이터는 관계(table)로 구성, 순서 상관 없는 튜플(row) 모음 (트랜잭션, 일괄처리 등 비즈니스 데이터 처리)

### 객체 관계형 불일치

RDB: 객체와 테이블 사이 인피던스 불일치* (ORM은 전환 계층의 boilerplate를 줄이나 완벽히 숨기지는 못함)

- 일대다 관계를 RDB에서 표현하는 방법
    1. 테이블 분리 후 외래키
    2. JSON이나 XML로 부호화 후 텍스트 칼럼에 저장, 애플리케이션에서 구조 해석 (질의 불가능, 지역성이 높다. 복잡한 조인 필요없이 하나의 질의로 조회 가능)

### NoSQL의 탄생

- 대규모 데이터, 쓰기 작업이 많을 때 확장성
- RDB 스키마에 비해 자유로운 데이터 모델링

### 다대일과 다대다 관계

- ID 저장 vs 문자열 저장
    - ID: 레코드에서 ID를 참조해 중복 X (RDB 정규화, 데이터는 점차 상호 연결되는 경향)
    - 문자열: 사용하는 모든 레코드에서 중복, 값 변경 시 모든 레코드 수정 필요 (NoSQL는 다대일, 다대다 관계 표현에 적합 X)

### 문서 데이터베이스는 역사를 반복하고 있나?

- 네트워크 모델: 관계를 포인터와 같이. 최상위 레코드로부터 연결 경로를 따름 (접근 경로)
- 관계형 모델: 관계(table)은 튜플(row)의 집합. 접근 경로 없이 조회 및 삽입 가능
    - Query optimizer: 질의 순서 및 사용 색인을 자동으로 결정 (접근 경로를 자동으로)
- 문서 데이터베이스: 일대다 관계를 중첩된 레코드로 표현 (별도 테이블 X)
    - 그러나 외래 키 역할을 하는 문서 참조로 다대일, 다대다 표현

### RDB와 오늘날의 문서 데이터베이스

- 문서 데이터베이스
    - 스키마 유연성 (읽기 스키마: 쓰기 시점 필드 존재 여부 보장 X), 지역성으로 나은 성능(한 번에 문서의 많은 부분 필요), 애플리케이션 데이터 구조와 유사
    - 다대다 관계 비정규화 데이터 일관성 유지 위해 추가 작업 필요
- 관계형 데이터베이스
    - 조인, 다대다 관계 유리
    - 샤딩*으로 복잡한 애플리케이션 코드, 스키마 변경 시 마이그레이션 필요, 조회 시 다중 조인으로 많은 디스크 탐색
- 관계 유형에 따라 적절한 데이터 모델 존재
    - 상호 연결이 많다? 그래프 > 관계형 > 문서
- 통합
    - 관계형 데이터베이스: JSON, XML 저장 및 질의 기능 제공
    - 문서 데이터베이스: 조인 지원 (클라이언트 조인 ⇒ 네트워크 비용 발생)

## 데이터를 위한 질의 언어

명령형 코드: 특정 순서로 특정 연산 (ex. Javascript DOM API)

선언형 질의(SQL): 목표 달성 방법보다는 알고자하는 데이터 조건 ⇒ DB 엔진 상세 구현을 숨김 (ex. CSS 선택자)

### 맵리듀스 질의

대량의 데이터를 클러스터 환경에서 분산 실행 및 처리하기 위한 프로그래밍 모델 ([https://mangkyu.tistory.com/129](https://mangkyu.tistory.com/129)), 선언형과 명령형 중간

- MongoDB의 맵리듀스
    - 질의와 일치하는 모든 문서에 대해
    - map 함수로 key-value 값 방출
    - key로 그룹화해 reduce 함수 호출

## 그래프형 데이터 모델

다대다 관계가 많을 때 유리 (모든 문서가 잠재적으로 관련 있다)

정점(vertex, 서로 다른 유형일 수도) + 간선(edge)

- 속성 그래프
- 사이퍼 질의 언어
- SQL의 그래프 질의
- 트리플 저장소와 스파클
- 초석: 데이터로그