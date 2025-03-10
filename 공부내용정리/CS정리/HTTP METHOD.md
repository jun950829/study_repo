# HTTTP METHOD

HTTP 메서드의 종류는 다음과 같으며 상황에 따라 유연하게 사용할 수 있습니다.

- GET : 리소스 조회. GET 메서드는 데이터를 가져올때만 사용.
- POST : 서버로 데이터를 전송한다. 새로운 리소스를 생성(등록)할 때 주로 사용.
- PUT : 요청 데이터를 사용하여 새로운 리소스를 생성하거나, 대상 리소스를 나타내는 데이터를 대체.
- PATCH : 리소스를 부분적으로 변경.
- DELETE : 리소스 삭제.

### PUT 과 PATCH 의 차이점
member/100 리소스에 "username" : "Lee", "age" : 20 이라는 데이터가 저장되어있다고 가정해보겠습니다. 위와 같이 PATCH 메서드로 해당 리소스 경로에 "age" : 50 데이터를 보내면 "age" 필드의 값만 변경되기 때문에 최종적으로 "username" : "Lee", "age" : 50 의 데이터를 가지게 됩니다.
반면 PUT 메서드는 방금과 같은 상황에서 기존 리소스를 완전히 대체하기 때문에 "age" : 50 의 데이터만 남게 됩니다. 기존 "username" 필드는 삭제된 것입니다.


### 좋은 API URL 설계❗🧨
여기서 오해할 수 있는 부분이 "HTTP 메서드 자체에는 기능이 없다"는 점입니다. 예를 들어, DELETE로 클라이언트에서 서버로 리소스 경로를 보내주면 자동으로 해당 데이터가 삭제되는 것이 아닙니다. 개발자가 그렇게 동작하도록 구현해주어야 하죠. 서버에서 DELETE 메서드 요청을 통해 신규 리소스가 생성되게 구현하면 그렇게 동작하고, 반대로 POST 메서드 요청을 통해 리소스를 삭제하게 만들수도 있습니다.
그럼 HTTP 메서드는 왜 필요할까요? 바로 리소스(자원)와 행위를 분리하기 위해서입니다.
좋은 API URL 설계는 리소스와 행위가 분리되어있어야 합니다. 즉, URL에 행위가 포함되어있으면 좋은 설계가 아니라는 뜻입니다. 예를 들어 회원 정보 관리 API URL를 설계한다고 가정해보겠습니다.

회원 목록 조회 : /read-member-list
회원 조회 : /read-member-by-id
회원 등록 : /create-member
회원 수정 : /update-member
회원 삭제 : /delete-member
다음과 같이 URL에 리소스(자원)와 행위가 포함되어있는 것 보다,

회원 목록 조회 : /members - GET
회원 조회 : /members/{id} - GET
회원 등록 : /members/{id} - POST
회원 수정 : /members/{id} - PATCH & PUT
회원 삭제 : /members/{id} - DELETE
위의 URL 처럼 리소스만 깔끔하게 보여지고 행위는 HTTP method로 구분하게끔 설계하는 것이 유지보수측면에서 훨씬 유리합니다. 이러한 URL 설계에 대한 더 많은 설명은 'REST api'에 대한 내용을 찾아보시면 됩니다.

