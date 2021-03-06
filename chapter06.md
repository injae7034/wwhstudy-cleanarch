# 만들면서 배우는 클린 아키텍쳐 6장 정리
# 영속성 어댑터 구현하기
## 의존성 역전
애플리케이션 서비스에서는 영속성 기능을 사용하기 의해 포트 인터페이스를 호출합니다.  

이 포트 인터페이스를 구현한 것이 바로 영속성 어댑터입니다.  

그래서 애플리케이션 서비스에서 포트 인터페이스를 호출하면 실제로는 영속성 어댑터가 호출되어 영속성 작업을 수행합니다.  

하지만 이 과정에서 애플리케이션 서비스는 영속성 어댑터에 대해서는 알 필요가 없고, 의존할 필요없이 오로지 포트 인터페이스만 알고 의존하면 됩니다.  

영속성 어댑터는 애플리케이션의 의해 호출되기 때문에 '주도되는' 아웃고잉 어댑터입니다.  

반대로 영속성 어댑터는 애플리케이션을 호출하지는 않습니다.  

애플리케이션 서비스는 포트 인터페이스를 호출하고 실제로는 영속성 어댑터가 호출되기 때문에 이 포트 인터페이스는 애플리케이션 서비스와 영속성 코드 사이를 이어주는 역할을 수행합니다.  

이렇게 하는 이유는 도메인 코드를 개발할 때 영속성 문제에 대해서는 신경쓰고 싶지 않고, 영속성 계층에 대한 코드 의존성을 없애기 위해서입니다.  

이렇게 되면 영속성 어댑터의 코드가 변경되거나 영속성 어댑터 전체가 바뀌더라도 애플리케이션 서비스는 포트 인터페이스만 의존하고 있기 때문에 코드를 변경할 필요가 없습니다.  

## 영속성 어댑터의 책임
영속성 어댑터가 하는 일은 다음과 같습니다.
<ol>
    <li>입력을 받는다</li>
    <li>입력을 데이터베이스 포맷으로 매핑한다</li>
    <li>입력을 데이터베이스로 보낸다</li>
    <li>데이터베이스 출력을 애플리케이션 포맷으로 매핑한다</li>
    <li>출력을 반환한다</li>
</ol>

### 1. 입력을 받는다
애플리케이션 서비스에서 포트 인터페이스를 호출하는데, 이 포트 인터페이스를 통해 영속성 어댑터가 필요한 입력 모델을 받습니다.  

이 때 이 입력 모델은 도메인 엔티티일수도 있고, 특정 데이터베이스 연산 전용 객체가 될 수도 있습니다.  

### 2. 입력을 데이터베이스 포맷으로 매핑한다
영속성 어댑터는 내부에서 데이터베이스를 쿼리하고나 변경하는데 사용할 수 있는 포맷으로 입력 받은 모델을 매핑합니다.  

자바에서는 일반적으로 데이터베이스와 통신할 때 JPA를 사용하기 때문에 입력 받은 모델을 데이터베이스 테이블 구조를 반영한 JPA 엔티티 객체로 매핑합니다.  

경우에 따라서는 입력 모델을 JPA 엔티티로 매핑하는 데 드는 노력에 비해 얻는 것이 많지 않을 경우 매핑하지 않는 전략을 사용할 수도 있습니다.  

JPA나 또는 다른 ORM, 데이터베이스와 통신하기 위한 어떤 기술을 사용해도 상관없습니다.  

중요한 것은 영속성 어댑터의 입력 모델이 영속성 어댑터 내부에 있는 것이 아니라 애플리케이션 코어에 있기 때문에 영속성 어댑터의 코드를 변경하더라도 코어에는 영향을 끼치지 않는다는 것입니다.  

### 3. 입력을 데이터베이스로 보낸다
위의 과정을 거쳐 영속성 어댑터는 데이터베이스에 쿼리를 날립니다.  

### 4. 데이터베이스의 출력을 애플리케이션 포맷으로 매핑한다
영속성 어댑터는 데이터베이스로부터 쿼리 결과를 받아서 포트 인터페이스에서 애플리케이션 서비스에 반환하기로 정의한 출력 모델로 매핑합니다. 

### 5. 출력을 반환한다
영속성 어댑터는 매핑된 출력 모델을 반환합니다.  

포트 인터페이스를 통해 이 출력 모델을 애플리케이션 서비스 계층에 반환합니다.  

애플리케이션 서비스 계층은 포트 인터페이스에 입력 모델을 넣고, 출력 모델을 반환받는데 실제로 내부에서는 누가 이를 구현하여 어떠한 일이 일어나는지 알 수 없고, 알 필요도 없습니다.  

즉, 입출력 모델이 영속성 어댑터에 있는 것이 아니라 애플리케이션 서비스 계층에 있다는 것입니다.  

## 포트 인터페이스 나누기
데이터베이스와 통신에 필요한 모든 메소드를 가지는 하나의 포트(리포지토리) 인터페이스를 만드는 것이 일반적인 방법입니다.  

그러나 앞에서 우리는 서비스 계층을 나눌 때 하나의 만능 서비스를 두지 않고, 세분화 하여 하나의 작은 행동만을 담당하는 서비스를 만들었습니다.  

예를 들어, 책에서는 AccountService라는 클래스에 sendMoney, registerAccount와 같은 메소드들을 추가하는 대신에 SendMoneyService, RegisterAccountService라는 클래스를 각각 만들었습니다.  

이처럼 서비스 계층은 각각의 메소드들을 모두 클래스화하여 세분화하였는데 하나의 Repository를 사용해 공유한다면 문제가 발생할 수 있습니다.  

예를 들어, Account의 정보만 구해오는 세분화된 서비스의 경우 데이터베이스에서 데이터를 구해오는 연산만 있으면 되는데, 공통의 Repository를 사용한다면 데이터베이스에 데이터를 새로 등록하거나 지우는 연산까지도 가지게 됩니다.  

그리고 모든 세분화된 서비스 계층이 공통의 repository를 공유하기 때문에 모든 세분화된 서비스 계층에서 이와 같은 문제가 발생합니다.  

즉, 각 세분화 된 서비스 계층들은 repository에서 자신이 사용하지 않는 메소드에도 의존성을 가지게 됩니다.  

이러한 의존성은 코드가 복잡해졌을 때 코드를 이해하기 힘들게 만들고, 단위 테스트를 할 경우 어떤 메서드에 모킹을 해야할지 정하는데 신경을 쓰게 만들어 테스트하기 어렵게 만듭니다.  

로버트 마틴은 다음과 같이 말했습니다.  

**"필요없는 화물을 운반하는 무언가에 의존하고 있으면 예상하지 못했던 문제가 생길 수 있다."**  

### 인터페이스 분리 원칙
이 원칙을 사용하면 세분화된 서비스 계층은 하나의 넓은 공용 레포지토리 인터페이스를 사용하는 것이 아니라 세분화된 서비스 계층에 특화된 여러 개의 전용 인터페이스로 나누게 됩니다.  

이렇게 되면 위에서 하나의 공용 레포지토리를 사용할 때 발생했던 불필요한 메소드에 대한 의존성이 사라지게 됩니다.  

왜냐하면 세분화된 서비스는 이제 진짜로 필요한 메소드만 가지는 특화된 레포지토리에 의존하기 때문입니다.  

더 나아가서 이렇게 특화된 레포지토리를 만들게 되면, 이 특화된 레포지토리는 대개 하나의 메소드만 가지기 때문에 이름을 지을 때 그 의도를 명확하게 표현할 수 있는 이점도 있습니다.  

이렇게 매우 세분화된 포트 인터페이스를 만들어두면 서비스 코드를 짤 때 자신이 필요한 포트에 그저 연결만 하면 되고, 거기서 어떤 메소드를 사용하고, 어떤 메소드를 안 쓰는지 구분할 필요가 없습니다.  

## 영속성 어댑터 나누기
이렇게 포트 인터페이스를 나누게 되면 당연히 이를 구현하는 영속성 어댑터도 하나일 필요가 없고, 세분화된 포트 인터페이스에 맞게 각각 구현하는 클래스를 만들면 됩니다.  

여기서 영속성 어댑터를 훨씬 더 잘게 나눌 수 있습니다.  

예를 들어 하나의 영속성 포트의 일부분을 구현하는 영속성 어댑터를 구현하고, 나머지 부분은 다른 영속성 어댑터가 구현하는 것도 가능합니다.  

여기서도 중요한 점은 도메인 코드는 영속성 포트에 의해 선언된 명세(메소드)를 어떤 영속성 어댑터가 구현하는지는 관심이 없다는 것입니다.  

그래서 영속성 포트를 하나의 영속성 어댑터가 구현을 하든지 두개의 영속성 어댑터가 구현하든지 도메인 코드와는 상관이 없습니다.  

애그리거트(불변식을 만족해서 하나의 단위로 취급될 수 있는 연관 객체의 모음)당 하나 또는 하나 이상의 영속성 어댑터 접근 방식은 여래 개의 bounded context의 영속성 요구사항을 분리할 때 즣은 이점으로 작용합니다.  

bounded context는 경계를 암시합니다.  

하나의 서비스가 다른 맥락에 있는 서비스의 영속성 어댑터에 직접 접근하지 않고, 필요하다면 전용 인커밍 포트를 통해 접근해야 합니다.  

## 스프링 데이터 JPA 예제
스프링 부트와 스프링 데이터를 이용하면 리포지토리 포트 인터페이스를 구현하는 구현체 없이, 즉, 개발자가 이를 구현하는 구현체를 별로도 만들 필요없이 포트 인터페이스에 명세된 메소드를 바로 사용할 수 있습니다.  

책의 예제에서 든 Account 클래스는 getter와 setter만 가진 간단한 데이터 클래스가 아니라 최대한 불변성을 유지하려고 합니다.  

유효한 상태의 Account 엔티티만 생성할 수 있는 팩터리 메서드를 제공하고, 출금 전에 계좌의 잔고를 확인하는 일과 같은 유효성 검증을 모든 상태 변경 메소드에서 수행하고 있기 때문에 유효하지 않은 도메인 모델이 생성되는 것을 막고 있습니다.  

또한 책에서 든 예제에서는 데이터베이스와 통신하기 위해 스프링 데이터 JPA를 사용하는데 이 때 데이터베이스의 상태를 표현하는 Entity 애너테이션을 도메인 엔티티에서 사용하지 않습니다.  

대신에 도메인 엔티티 클래스와 별도인 JPA 전용 클래스를 추가적으로 생성하고 이를 도메인 엔티티와 매핑시킵니다.  

이렇게 구분한 후에 나중에 따로 매핑하는 이유는 JPA 애노테이션을 도메인 모델에 사용하게 되면 도메인 모델이 JPA에 영향을 받기 때문입니다.  

예를 들어, 도메인 모델에서는 기본 생성자가 필요없는데 JPA 엔티티로 인해 기본 생성자를 별도로 정의해줘야 합니다.  

이처럼 영속성 계층에 영향을 받지 않는 순수한 도메인 모델을 만들고 싶다면 별도의 영속성 모델을 만들고 이를 도메인 모델과 매핑하는 것이 좋습니다.  

## 데이터베이스 트랜잭션은 어떻게 해야 할까?
트랜잭션은 하나의 특정한 유스케이스에 대해 일어나는 모든 쓰기 작업에 걸쳐 있어야 합니다.  

그래야 그 중에 하나라도 실패할 경우 다 같이 롤백을 할 수 있기 떄문입니다.  

영속성 어댑터는 어떤 데이터베이스 연산이 같은 유스케이스에 포함되는지 알지 못하기 때문에 언제 트랜잭션을 열고 닫을지 결정할 수 없습니다.  

그래서 이 책임은 양속성 어댑터를 호출하는 서비스에게 위임해야 합니다.  

자바 스프링에서 가장 쉬운 방법은 Transactional 애너테이션을 애플리케이션 서비스 클래스에 붙여서 스프링이 모든 publiv 메서드를 트랜잭션으로 감싸게 하는 것입니다.  

## 유지보수 가능한 소프트웨어를 만드는 데 어떻게 도움이 될까?
세분화된 애플리케이션 서비스 코드에 세분화된 전용 포트 인터페이스, 그리고 세분화된 전용 영속성 어탭터를 구현하게 되면 도메인 코드와 영속성 코드가 분리도이서 영속성 어댑터의 코드가 변경되더라도 도메인 코드에는 영향을 끼치지 않습니다.  

또한 각각의 포트와 어댑터가 세분화되었기 때문에 명칭을 세분화해서 지을 수 있고, 이것이 코드를 이해하고 유지 보수 관리하는 것에 큰 도움이 됩니다.  

또한 세분화된 포트 인터페이스를 사용하면 각 포트와 인터페이스들은 서로 의존하지 않고, 불필요한 코드가 없으며 포트 인터페이스마다 다른 기술이나 방식으로 구현할 수 있는 유연함이 생깁니다.  

그리고 이 모든 과정에서 도메인 코드는 전혀 영향을 받지 않는다는 것이 핵심입니다.  
