## 4장 부호화와 발전
- 순회식 업그레이드(rolling upgrade, 단계적 롤아웃 staged rollout) : 서버측 어플리케이션에서 한번에 몇개 의 노드에 새 버전을 배포하고 새로운 버전이 원할하게 실행되는지 확인한 다음 서서히 모든 노드에 실행되게 하는 방식
- 호환성:
  - 하위 호환성 : 새로운 코드는 예전 코드가 기록된 데이터를 읽을 수 있어야 한다.
  - 상위 호환성 : 예전 코드는 새로운 코득 ㅏ기록한 데이터를 읽을 수 있어야 한다.

- 주요 키워드 : JSON, XML, Protocol Buffer, Thrift, Avro, REST, RPC, Message Queue

### 데이터 부호화 형식
- 부호화(encoding), 직렬화(serialization), 마샬링(marshaling)
- 복호화(decoding), 역직렬화(deserialization), 언마샬링(unmarshaling)

#### 언어별 코드
- JAVA:
  - 참고 : [직렬화(Serialization)란 무엇일까?](https://devlog-wjdrbs96.tistory.com/268), [Serializable Javadoc](https://docs.oracle.com/javase/7/docs/api/java/io/Serializable.html)

  - 정의
  ```java
  private void writeObject(java.io.ObjectOutputStream out)
   throws IOException
  private void readObject(java.io.ObjectInputStream in)
   throws IOException, ClassNotFoundException;
  private void readObjectNoData()
   throws ObjectStreamException;
  ```

  - 사용법
    - 읽기
      ```java
      FileInputStream fis = new FileInputStream(fullPath);
      ObjectInputStream ois = new ObjectInputStream(fis);
      Object obj = ois.readObject();
      SerialDTO dto = (SerialDTO)obj;
      ```
    - 쓰기
      ```java
      SerialDTO dto;
      FileOutputStream fos = new FileOutputStream(filePath);
      ObjectOutputStream oos = new ObjectOutputStream(fos);
      oos.writeObject(dto);
      ```

- python
  - 찾아보니 python에는 marshal, pickle 라이브러리 둘다 있는데 일반적으로 pickle 이 더 선호된다고 합니다.
  - 참고 : [python docs-pickle](https://docs.python.org/ko/3/library/pickle.html)
  - 사용법
    - 읽기 :
      ```python
      with open('list.txt', 'wb') as f:
         pickle.dump(list, f)
      ```
    - 쓰기 :
      ```python
      with open('list.txt', 'rb') as f:
         data = pickle.load(f)
      ```

- go
  - [go.dev - gob](https://pkg.go.dev/encoding/gob)
  - 사용법:
    - 읽기
      ```go
      var buf bytes.Buffer
      dec := gob.NewDecoder(&network)
      var obj Obj
      err = dec.Decode(&obj)
      if err != nil {
        log.Fatal("decode error:", err)
      }
      ```
    - 쓰기
      ```go
      var buf bytes.Buffer
      enc := gob.NewEncoder(&buf)
      err := enc.Encode(obj)
      if err != nil {
        log.Fatal("encode error:", err)
      }
      ```

#### 프로그래밍 언어에 내장된 부호화 라이브러리의 장단점
- 장점 : 편리함
- 단점 :
  - 언어에 종속적이다.
  - 보안에 취약하다. (일반적인 모든 부호화의 문제점)
  - 데이터 버전관리에 문제가 있다.(데이터 버전간 호환성이 떨어진다.)
  - 효율성(자바의 직렬화는 안좋기로 소문이 나있다.)

### JSON과 XML, 이진 변형





---
### 이야기 거리
- [line 의 Antman 프로젝트 개발기](https://engineering.linecorp.com/ko/blog/antman-project-development-story/)
