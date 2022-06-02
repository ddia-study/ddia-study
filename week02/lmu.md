## 2 장 데이터 모델과 질의 언어
- 관계형 모델과 문서모델
  - 관계형 모델의 장점 : 비지니스 데이터 처리 (트랜잭션, 일괄처리)
  - NoSQL의 장점:
    - 높은 쓰기 처리량(대규모 데이터셋에 유리)
    - 무료 오픈 소스 소프트웨어
    - 특수한 연산 지원
    - 동적이고 풍부한 표현력
  - 다중 저장소 지속성 : 관계형 데이터베이스와 비관계형 데이터 스토어가 함께 사용되는 개념
  - *개인 생각* : Web과 REST API의 선호도 증가에 따라 개발자에게 친숙해진 영향도 있다

- Q&A 시간
---
- 객체 관계형 불일치
  - 임피던스 불일치(impedance mismatch) : 객체-관계 간 모델 불일치, 상속 불일치, 관계와 연관 관계의 불일치
    - [참고](http://www.libqa.com/wiki/769)
  - 언제나 한번에 조회되는 것도 아니고, 중복제거 등 다양한 문제가 섞여있음.
  - 예시:
    - django 에서 many-to-many 관계
        ```python
        # models.py
        from django.db import models

        class Publication(models.Model):
            title = models.CharField(max_length=30)

            class Meta:
                ordering = ['title']

            def __str__(self):
                return self.title

        class Article(models.Model):
            headline = models.CharField(max_length=100)
            publications = models.ManyToManyField(Publication)

            class Meta:
                ordering = ['headline']

            def __str__(self):
                return self.headline

        # usage.py
        p1 = Publication(title='The Python Journal')
        p1.save()
        p2 = Publication(title='Science News')
        p2.save()
        p3 = Publication(title='Science Weekly')
        p3.save()

        # link
        a1 = Article(headline='Django lets you build web apps easily')
        a1.publications.add(p1)

        # filter
        Publication.objects.filter(article=1)
        ```
        - [stackoverflow:Many to many lookups in Django](https://stackoverflow.com/questions/1318366/many-to-many-lookups-in-django)
    - jpa 에서 many-to-many 관계
        - [코드 출처](https://happyer16.tistory.com/entry/Spring-JPA-%EB%8B%A4%EB%8C%80%EB%8B%A4-%EC%84%A4%EC%A0%95-%EB%B0%8F-%EC%A3%BC%EC%9D%98-Many-To-Many)
        ```java
        @Entity
        class Student {

            @Id
            Long id;

            @ManyToMany
            @JoinTable(
              name = "course_like",
              joinColumns = @JoinColumn(name = "student_id"),
              inverseJoinColumns = @JoinColumn(name = "course_id"))
            Set<Course> likedCourses;

            // additional properties
            // standard constructors, getters, and setters
        }

        @Entity
        class Course {

            @Id
            Long id;

            @ManyToMany(mappedBy = "likedCourses")
            Set<Student> likes;

            // additional properties
            // standard constructors, getters, and setters
        }
        ```
        - [[JPA] 즉시 로딩과 지연 로딩(FetchType.LAZY or EAGER)](https://ict-nroo.tistory.com/132)

    - gorm 에서 many-to-many 관계
        ```go
        // User has and belongs to many languages, `user_languages` is the join table
        type User struct {
          gorm.Model
          Languages []Language `gorm:"many2many:user_languages;"`
        }

        type Language struct {
          gorm.Model
          Name string
        }
        ```

- 다대일과 다대다 관계
  - 관계형이 강점을 보이는 이유 : 조인연산
  - 다대일(many-to-one)은 조인을 흉내내야한다.
  - 다대다(many-to-many)는 조인 연산이 없다면 사실상 불가능하다

- Q&A 시간
---

- 문서 데이터베이스는 역사를 반복하고 있나?
  - 네트워크 모델
    - 개인 추가 자료
      - https://github.com/twitter-archive/flockdb
      - 과거 twitter 에서도 graph 기반 모델을 사용해서 개발한 전적이 있으나 현재는 RDB와 Document DB 등을 섞어서 사용하는 것으로 알고 있습니다.
  - 관계형 모델
- 관계형 데이터베이스와 오늘날의 문서 데이터베이스
  - 문서 데이터 모델의 선호 이유
    - 스키마 유연성, 지역성
    - 일부 어클리케이션의 데이터 구조와 유사
  - 관계형의 선호 이유
    - 조인, 다대일, 다대다 관계 지원

- 어떤 데이터 모델이 애플리케이션 코드를 더 간단하게 할까?
  - 아직 모름
  - *개인 생각* : 발표 이후 스터디에서 선호도 및 자신의 사례를 공유하면서 이야기를 나눠봤으면 함

- Q&A 시간
---

- 문서 모델에서의 스키마 유연성
  - 읽기 스키마(schema-on-read)
  - 쓰기 스키마(schema-on-write)
  - 문서 데이터베이스에서 종종 불리는 schemaless는 쓰기 스키마를 말하는 것이다.
  - 일반적인 RDB에서는 마이그레이션(migration)을 수행해서 스키마를 변경하게 된다.
    - *개인 생각* : 마이그레이션의 생산성 및 성능에 대해서 토론하면 좋겠습니다.
        - [django migration 관련 글](https://tibetsandfox.tistory.com/24)
        - jpa:
          ```properties
          spring.jpa.hibernate.ddl-auto=create
          spring.jpa.generate-ddl=false
          ```
          - [JPA migration 관련 글](https://wooody92.github.io/spring%20boot/Spring-Boot-DB-%EC%B4%88%EA%B8%B0%ED%99%94%EC%99%80-DB-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98-%ED%88%B4/)

- Q&A 시간
---

- 질의를 위한 데이터 지역성
  - 저장소 지역성(storage locality)를 잘 살리려면 문서를 작게 유지하는 것을 권장한다
  - 다중 테이블 색인 클러스터 테이블, 칼럼 패밀리 개념 등

- 문서 데이터베이스와 관계형 데이터베이스의 통합
  - MySQL 5.7, PostgreSQL 9.3부터 JSON 문서를 잘 지원한다.

- 데이터를 위한 질의언어
  - 수식 : ![formula](https://render.githubusercontent.com/render/math?math=sharks=\sigma_{\text{family="Sharks"}}(animals))
  - sql :
  ```sql
  SELECT * FROM animals WHERE family = 'Sharks';
  ```

  - 선언형 질의 언어 : 간결함, 성능향상 기대, 병렬처리에 적합

- 웹에서의 선언형 질의
  - css, XSL

- 맵리듀스 질의
  - 맵리듀스(MapReduce)
    - 참고 자료 : https://12bme.tistory.com/154
    - 논문 : https://static.googleusercontent.com/media/research.google.com/ko//archive/mapreduce-osdi04.pdf

- Q&A 시간
---

- 그래프형 데이터 모델
    - 속성 그래프 모델, 트리플 저장소 모델
    - 그래프용 선언형 질의 언어 : 사이퍼, 스파클, 데이터로그
    - 데이터 로그
    - 그래프 데이터 베이스 vs 네트워크 모델

    - 이 부분은 잘 와닿지 않고, 굳이 많이 언급할 필요가 없다고 생각해서 생략했습니다.
    - 만약, 이야기할 거리가 있다면 책으로 하면 좋겠습니다.



- 별도 참고자료 :
  - AWS 에서 지원하는 Graph Database:
    - [Amazon Neptune](https://docs.aws.amazon.com/neptune/latest/userguide/graph-get-started.html)

  - GraphQL:
    - [GQL은 왜 핫할까?](https://velog.io/@mnjsk7541/GQL-%ED%95%AB%ED%95%9C-%EC%9D%B4%EC%9C%A0)
    - [GraphQL, Facebook이 꿈꾼 새로운 패러다임](https://uzihoon.com/post/82746960-ed3c-11eb-a1b6-81da1e62e780)

---

