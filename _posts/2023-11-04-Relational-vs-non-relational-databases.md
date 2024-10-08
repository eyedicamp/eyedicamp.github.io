---
layout: post
title: "관계형 vs 비관계형 데이터베이스"
date:   2023-11-04
excerpt: "관계형 DB와 비관계형 DB에 관하여"
tag: 
 - AI Expert Roadmap
 - Fundamentals
 - Database
comments: true
---

### DB는 왜 중요한가?

좋은 모델을 만들기 위해서는 좋은 데이터가 필요하다. 데이터의 양과 질도 중요하지만,데이터를 어떤 형식으로 저장하는지 또한 효율 측면에 큰 영향을 미친다. 첫 게시물은 Database Basics, 그 중에서도 "관계형 DB와 비관계형 DB에 관하여" 이다.
  
  
  

### 먼저, 관계형 데이터베이스는?

관계형 데이터베이스는 한 단어로 표현할 수 있다.
  

#### "SQL"
  

SQL은 Structured Query Language라는 뜻으로, 한글로 말하자면 구조화 된 쿼리 언어이다. SQL은 관계형 데이터베이스 자체를 뜻하는 말은 아니고, 관계형 데이터베이스를 만들고 관리하기 위해 사용하는 언어이다. 따라서 역으로 관계형 데이터베이스는 SQL로 만들어진 DB인 것이다.
  
  
  

### 그래서 관계형 데이터베이스는 어떤데?

관계형 DB는 엄격한 스키마(schema), 즉 구조로 이루어져있다. 관계형 DB에서의 스키마는 table로 이루어져있고, 따라서 관계형 DB는 고정된 row와 column을 가진 표(table)에만 작성될 수 있다. 그리고 데이터들은 관계를 통해서 여러 테이블에 분산되어있다.
  
  
  

### 관계?

관계란, 데이터를 중복으로 작성하는 것을 방지하기 위해 사용하는 방식이다. 예를 들어 아래 표를 보면 표를 하나로 작성하면 order에 대한 정보가 양쪽 표에 모두 들어가서, 데이터를 중복 저장하여 메모리 손해를 볼 수 있는데, user_id와 product_id를 일종의 포인터처럼 활용해서 중복을 피한다. 이를 통해 데이터를 수정하는 경우에 발생할 수 있는 동기화 오류 또한 방지할 수 있다.

![table about relation]({{site.baseurl}}/assets/img/Relational-vs-non-relational-databases/table1.jpg)
  
  
  

### 데이터는 원래 표로 표현하는거 아니야? NoSQL은 그럼 뭔데?

NoSQL은 Not Only SQL이라는 뜻으로, 이름 그대로 SQL과 같은 table 형태만을 가지는 것이 아니라, Key-Value 타입, 문서형 DB, Wide-Column Store DB, Graph DB 등 다양한 스키마로 데이터를 저장할 수 있다. 또한 다른 구조의 데이터를 하나의 table에 저장할 수 있다.

  
  
  
### 그럼 다양하게 쓸 수 있는 NoSQL이 무조건 좋은거 아니야?

세상이 다 그렇듯, 무조건 좋은건 없다. 아래는 SQL과 NoSQL의 장단점이다.


#### SQL 장점
* 스키마가 명확해서 데이터 무결성을 보장하기 좋다.


#### SQL 단점
* 이미 작성된 스키마 수정 어렵다. (DB 구조 변경 불가)
* JOIN이 굉장히 많아져, 직관적이지 않은 쿼리문이 생길 수 있다.
* 데이터가 너무 커지면 읽기, 쓰기가 비효율적이다.


#### NoSQL 장점
* 유연하다. (이미 저장된 데이터 조정, 새로운 필드 추가 등)
* 저장방법을 최적화하여 효율을 높일 수 있다.
* 복잡한 데이터구조 표현이 가능하다.


#### NoSQL 단점
* 데이터를 변경 할 경우 여러 곳에서 수정을 해야하므로 효율이 떨어진다.

  
  
  
### 그래서 뭐 쓰지?

데이터 신뢰성이 높아야 할 때, 구조가 단순하고 많이 바뀌지 않는 경우엔 SQL, 데이터 형태가 복잡하거나 양이 많을때, 읽기/쓰기를 많이 하는 경우엔 NoSQL을 쓰는게 좋다.


#### SQL의 종류
* MySQL
* PostgreSQL
* SQLite


#### NoSQL의 종류
* Redis
* Cassandra
* HBase
* Firebase
* MongoDB




### 결론
SQL등의 DB관리에 대한 부분은 백엔드 개발자가 공부해야할 부분이라고 생각하여 공부를 안하고있었는데, 좀 더 관심을 가지고 개인 프로젝트를 할 때도 체계적으로 DB를 활용하는 등 써먹어볼 방식을 생각해봐야겠다.






### 출처, 더 궁금하다면?
* [[DB]SQL(관계형 데이터베이스)과 NoSQL(비 관계형 데이터베이스) 개념/비교/차이](https://devuna.tistory.com/25)
* [[데이터베이스] SQL(구조화 쿼리 언어) vs NoSQL(비구조화 쿼리 언어)](https://hanamon.kr/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-sql-vs-nosql/)
* [SQL vs NoSQL 5분컷 설명!](https://www.youtube.com/watch?v=Q_9cFgzZr8Q&t=73s)