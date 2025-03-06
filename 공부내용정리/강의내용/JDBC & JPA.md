# JDBC
자바 언어로 데이터베이스 프로그래밍을 하기위한 라이브러리

jdbc를 매번 로딩해줘야하고 예외처리를 너무 많이 설정 해주어야함 -> JPA가 나옴

# JPA

JPA란 JAVA ORM(Object Relational Mapping) 기술에 대한 인터페이스

ORM - 객체와 데이터베이스의 관계를 맵핑하는방법

EX)
```
    public class User {
        private String name;
        private int age;
        private String email;
    }
```

# Hibernate
JPA의 인터페이스를 구현한 라이브러리

```
EntityManager entityManager = entityManager.getTransaction().begin();

var user = new User("hong", 20, "hong@gmail.com");

entityManager.persist(user);
entityManager.getTransaction().commit();
entityManager.close();
```

# Spring Data JPA
Hibernate 외에 어떠한 라이브러리를 써도 반복되는 작업의 발생때문에 이를 편리하게 하고 transaction관리도 spring 에서 관리해주는 형태


```
    @Transactional
    public User save(User user) {
        return userRepository.save(user);
    }
    
    @Transactional
    @Override
    public <S extends T> S save(S entity) {
        
        Assert.notNull(entity, "Entity must no be null.");
        
        if(entityInformation.isNew(entity) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }
    
```