# QueryDSL의 의존성은 어디에 추가해야 하는가

module-api과 module-domain 중 QueryDSL의 의존성은 어디에 추가되어야 하는가
얼핏보면 module-api가 맞는 것 같지만 아니다

`QueryDSL의 의존성은 module-domain이 맞다`

왜?
QueryDSL은 @Entity 클래스 기준으로 Q타입 클래스를 생성한다
그래서 엔티티 클래스들이 정의된 module-domain에 QueryDSL 관련 의존성이 추가되어야 한다

| 역할                   | 들어갈 모듈          | 이유                |
| -------------------- | --------------- | ----------------- |
| `@Entity` 정의         | `module-domain` | 도메인 객체 보관         |
| `Q클래스 생성`            | `module-domain` | Entity 기준이니까      |
| `JPAQueryFactory 사용` | `module-api`    | API 로직에서 동적 쿼리 작성 |