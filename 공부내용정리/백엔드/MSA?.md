# Microservice Architecture의 정의
- 비즈니스 도메인을 중심으로 서비스를 모델리하고 구현하는 아키텍처 스타일
  - vs monolithic: 하나의 프로젝트 구조 안에서 모든 도메인을 구현하는 방식
- 도메인 서비스 간의 통신은 네트워크 기반의 HTTP API 또는 비동기 메시징 방식 등으로 이루어짐
- 각 도메인 서비스는 자체 Datavase를 가짐


## Microservice Architecture 전환을 고려해야 하는 시점
- 생존을 걱정하던 초기 스타트업에서 벗어나 비즈니스 규모가 어느 정도 궤도에 오르는 시점
- monolithic 구조의 장점보다 단점이 부각되는 시점
  - 하나의 repository에 코드 베이스가 개개인이 감당할 수 없는 수준으로 커짐
  

## Microservice Architecture의 장점과 단점
- 우버가 msa를 도입한 이유 -> 확장성 개전
  - 증가하는 트래픽 처리
  - 새로운 기능을 쉽게 추가
  - 조직의 성장에 쉽게 적응할 수 있는 아키텍처 적용
- 장점
  - 각각의 비즈니스 도메인별로 독립적인 서비스를 운영할 수 있다
  - 빠른 구현과 배포가 가능해진다
  - 팀의 책임과 자율성이 극대화 된다
- 단점
  - 네트워크 기반의 API 호출로 서비스가 구성되기 때문에 프로세스간 통신에 비해 느리고 복잡하다.
  - 일관된 트랜잭션과 데이터 정합성을 유지하기 어렵다
  - 테스트와 장애추적, 모니터링 등이 쉽지 않다

    

