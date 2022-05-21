# 만들면서 배우는 클린 아키텍쳐 7장 정리
# 아키텍처 요소 테스트하기
이번 장에서는 육각형 아키텍처에서의 테스트 전략과 아키텍처의 각 요소들을 테스트할 수 있는 테스트 유형에 대해 이야기합니다.  

# 테스트 유형
책에서는 육각형 아키텍처를 테스트 하기 위해 단위 테스트, 통합 테스트, 시스템 테스트라는 용어를 사용하는데 이에 대한 정의는 맥락에 따라 다르며, 프로젝트마다 다른 의미를 가질 수 있습니다.  

# 테스트 피라미드
또한 맥락에 따라 테스트 피라미드에 포함되는 계층 역시 달라질 수 있습니다. 

테스트 피라미드에 따르면 비용이 많이 드는 테스트는 지양하고, 비용이 적게 드는 테스트를 많이 만들어야 합니다.  

테스트 피라미드에는 제일 밑에 층에 단위 테스트, 중간에 통합 테스트, 제일 상단에 시스템 테스트가 위치하며 밑에 층일수록 비용이 적게 들고 위에 층일수록 비용이 많이 듭니다.

피라미드의 위에 위치한 테스트는 일반적으로 여러 개의 단위와 단위를 넘는 경계, 아키텍처 경계, 시스템의 경계를 결합하는 테스트는 만드는 비용이 비싸며 실행이 더 느려지고, 기능 에러보다 설정 에러로 인해 깨지기 더 쉽습니다.  

이렇게 테스트가 비싸질수록 테스트의 커버리지 목표는 낮게 잡아야 합니다.  

그렇지 않으면 새로운 기능을 만드는 것보다 테스트를 만드는 데 시간을 더 사용하기 때문입니다.  

## 단위 테스트
테스트 피라미드의 가장 아래에 위치하여 토대에 해당하는 단위 테스트는 하나의 단위(일반적으로 하나의 클래스)가 제대로 동작하는지 확인할 수 있는 테스트입니다.  

이러한 단위 테스트는 만드는 비용이 적고, 유지보수하기 쉽고, 빨리 실행되고, 안정적인 작은 크기의 테스트들에 대해 높은 커버리지를 유지할 수 있습니다.  

일반적으로 하나의 클래스를 인스턴스화하고 해당 클래스의 인터페이스를 통해 기능들을 테스트 합니다.  

만약 테스트 중인 클래스가 다른 클래스에 의존한다면 테스트 중인 클래스에서 의존하는 클래스를 직접 인스턴스화하지 않고, 테스트하는 동안 필요한 작업을 흉내 내는 목(mock)으로 대체합니다.  

## 통합 테스트
테스트 피라미드에서 단위 테스트의 다음 계층은 통합테스트입니다.  

이 테스트는 연결된 여러 유닛을 인스턴스화하고 시작점이 되는 클래스의 인터페이스로 데이터를 보낸 후 유닛들의 네트워크가 기대한대로 잘 동작하는지 검증합니다.  

상황에 따라서 두 계층 간의 경계를 걸쳐서도 테스트 할 수 있기 때문에 객체 네트워크가 완전하지 않을 수 있고, 어떤 시점에는 목(mock)을 대상으로 수행해야 합니다.  

## 시스템 테스트
애플리케이션을 구성하는 모든 객체 네트워크를 가동시켜 특정 유스케이스가 전 계층에서 잘 동작하는지 검증합니다.  

## 엔드투엔드(end-to-end) 테스트
시스템 테스트 위에 위치하는 테스트로써 시스템 테스트에서 했던 것과 더불어 애플리케이션의 UI를 포함해 테스트합니다.

# 단위 테스트로 도메인 엔티티 테스트하기
육각형 아키텍처의 중심인 도메인 엔티티를 테스트하기 위해서는 단위 테스트가 적합합니다.  

책에서 사례로 제시한 Account 엔티티의 상태는 과거 특정 시점의 계좌 잔고(baselineBalance)와 그 이후의 입출금 내역(activity)로 구성되어 있습니다.  

이 Account 엔티티의 메소드 중 하나인 withdraw 메서드의 테스트 코드는 아래와 같습니다.  

```java
class AccountTest {
    @Test
    void withdrawalSucceeds() {
        AccountId accountId = new AccountId(1L);
		Account account = defaultAccount()
				.withAccountId(accountId)
				.withBaselineBalance(Money.of(555L))
				.withActivityWindow(new ActivityWindow(
						defaultActivity()
								.withTargetAccount(accountId)
								.withMoney(Money.of(999L)).build(),
						defaultActivity()
								.withTargetAccount(accountId)
								.withMoney(Money.of(1L)).build()))
				.build();

		boolean success = account.withdraw(Money.of(555L), new AccountId(99L));

		assertThat(success).isTrue();
		assertThat(account.getActivityWindow().getActivities()).hasSize(3);
		assertThat(account.calculateBalance()).isEqualTo(Money.of(1000L));
    }
}
```

위 코드는 특정 상태의 Account를 인스턴스화하고(given), withdraw 메서드를 호출(when)해서 출금이 성공했는지 boolean반환값을 통해 확인하고(then), Account 객체의 상태에 대해 기대되는 부수효과들이 잘 일어났는지 확인하는(then) 단순한 단위 테스트입니다.  

위와 같은 테스트는 만들고 이해하기 쉽고, 아주 빠르게 실행됩니다.  

도메인 엔티티의 행동은 다른 클래스에 거의 의존하지 않기 때문에 다른 종류의 테스트는 필요없으며 단위 테스트야말로 도메인 엔티티에 녹아 있는 비즈니스 규칙을 검증하기 가장 적절한 방법입니다.  

# 단위 테스트로 유스케이스 테스트하기
이제 육각형 아키텍처에서 도메인 엔티티 클래스에 대한 테스트를 마쳤으니 계층의 바깥쪽에 위치한 유스케이스를 테스트할 차례입니다.  

책에서는 SendMoneyService를 테스트 예시로 제시합니다.  

SendMoney 유스케이스는 출금 계좌 잔고가 다른 트랜잭션에 의해 변경도지 않도록 락(lock)을 겁니다.  

출금 계좌에서 돈이 출금되고 나면 똑같이 입금 계좌에 락을 걸고 돈을 입금시킵니다.  

이 과정이 끝나면 두 계좌에서 모두 락을 해제합니다.  

아래의 테스트 코드에서 트랜잭션이 성공했을 때 유스케이스대로 잘 동작하는지 테스트합니다.  

```java
class SendMoneyServiceTest {

    //필드 선언은 생략

    @Test
	void transactionSucceeds() {

		Account sourceAccount = givenSourceAccount();
		Account targetAccount = givenTargetAccount();

		givenWithdrawalWillSucceed(sourceAccount);
		givenDepositWillSucceed(targetAccount);

		Money money = Money.of(500L);

		SendMoneyCommand command = new SendMoneyCommand(
				sourceAccount.getId().get(),
				targetAccount.getId().get(),
				money);

		boolean success = sendMoneyService.sendMoney(command);

		assertThat(success).isTrue();

		AccountId sourceAccountId = sourceAccount.getId().get();
		AccountId targetAccountId = targetAccount.getId().get();

		then(accountLock).should().lockAccount(eq(sourceAccountId));
		then(sourceAccount).should().withdraw(eq(money), eq(targetAccountId));
		then(accountLock).should().releaseAccount(eq(sourceAccountId));

		then(accountLock).should().lockAccount(eq(targetAccountId));
		then(targetAccount).should().deposit(eq(money), eq(sourceAccountId));
		then(accountLock).should().releaseAccount(eq(targetAccountId));

		thenAccountsHaveBeenUpdated(sourceAccountId, targetAccountId);
	}

    //헬퍼 메서드는 생략
}
```

테스트 가독성을 높이기 위해 행동-주도 개발(behavior driven development)에서 일반적으로 사용하는 방식인 given/when/then을 사용합니다.  

## given
given에서 출금(source) 및 입금(target) Account의 인스턴스(sourceAccount, targetAccount)를 각각 생성하고, 적절한 상태로 만듭니다.(givenSourceAccount와 givenTargetAccount 메서드 호출)  

그 다음으로 이렇게 초기화 된 Account들의 인스턴스를 각각 givenWithdrawalWillSucceed와 givenDepositWillSucced 메소드의 매개변수로 넣습니다.  

입력 모델인 SendMoneyCommand 인스턴스를 생성하면서 출금과 입금의 Account 인스턴스를 매개변수로 넣어주는데 이를 통해 SendMoneyUsecase의 입력 유효성을 검증합니다.  

## when
when에서는 SendMoenyService의 sendMoney 메서드를 호출하면서 입력 유효성 검증이 끝난 SendMoneyCommand 인스턴스를 매개변수로 넣어줍니다.  

## then
then에서는 sendMoney의 반환값인 boolean이 true인지 확인하여 성공적으로 송금이 이루어졌는지 검증합니다.  

그리고 츨금 및 밉금 Account, 계좌에 락을 걸고 해제하는 책임을 가진 AccountLock에 대해 특정 메서드가 호출되었는지를 확인합니다.  

## Mockito 이용
SendMoneyServiceTest의 givenWithdrawalWillSucceed와 givenDepositWillSucceed 메서드는 Mockito 라이브러리 given을 이용해 목(mock)객체를 생성합니다.  

```java
class SendMoneyServiceTest {

    //생략

    private void givenWithdrawalWillSucceed(Account account) {
		given(account.withdraw(any(Money.class), any(AccountId.class)))
				.willReturn(true);
	}

    private void givenDepositWillSucceed(Account account) {
		given(account.deposit(any(Money.class), any(AccountId.class)))
				.willReturn(true);
	}

    //생략

}
```

또한 아까 transactionSucceeds 코드에서 출금 및 입급 Account와 계좌에 락을 걸고 해제하는 책임을 가진 AccountLock에 대해 특정 메서드가 호출되었는지 검증하기 위해 사용한 것이 바로 Mockito 라이브러리의 then입니다.  

## 유의사항
테스트 중인 유스케이스 서비스는 stateless(무상태)이기 때문에 'then'에서 특정 상태를 검증할 수 없습니다.  

즉, 도메인 엔티티인 Account 인스턴스에 무슨 값이 저장되어 있고, 무슨 값이 저장되야 맞고 틀린지를 확인할 수 없다는 말입니다.  

대신에 유스케이스 서비스 테스트에서는 Mockito 라이브러리 given을 이용해 생성된 목(mock)객체, 즉, 서비스가 의존하는 대상의 특정 매서드와 상호작용했는지 여부를 검증합니다.  

**이렇게 되면 테스트는 코드의 행동 변경뿐만 아니라 코드의 구조 변경에도 취약해집니다?(무슨 말인지 아직 이해못함)**

이 상황에서는 코드가 리팩터링되면 테스트도 변경해야할 확률이 높아집니다.  

그래서 테스트에서 어떤 상호작용을 검증하고 싶은지 신중하게 생각해야 합니다.  

따라서 앞에서 주어진 transactionSucceeds 테스트 코드처럼 **모든** 동작을 검증하는 것이 아니라 중요한 핵심 부분만 골라 집중해서 테스트하는 것이 바람직합니다.  

왜냐하면 모든 동작을 검증하려고 하면 클래스가 조금이라도 바뀔 때마다 테스트를 계속 변경해야 해서 테스트의 가치가 떨어지기 때문입니다.  

## 단위 테스트? or 통합 테스트? 
유스케이스의 테스트는 단위 테스트로 분류하였지만 의존성의 상호작용을 테스트 하고 있기 때문에 통합 테스트에 더 가깝습니다.  

하지만 실제 의존성을 관리하지 않고, Mockito 라이브러리를 사용해 목(mock)으로 작업하고 있기 때문에 완전한 통합 테스트에 비해서는 만들고 유지보수하기 쉽습니다.  

# 통합 테스트로 웹 어댑터 테스트하기
이제 유스케이스에서 한 계층 더 바깥으로 나가서 웹 어댑터를 테스트할 차례입니다.  

## 웹 어댑터에서 데이터 흐름
웹 어댑터는 HTTP를 통해 JSON 문자열 등의 형태로 데이터를 입력 받고, 입력에 대한 유효성을 검증하고, 유스케이스에서 사용할 수 있는 포맷으로 매핑하여 유스케이스에 데이터를 전달합니다.  

그리고 나서 유스케이스가 처리한 결과를 다시 JSON으로 매핑하고 HTTP 응답을 통해 클라이언트에 반환합니다.  

따라서 웹 어댑터 테스트에서는 앞의 모든 과정들이 기대한 대로 동작하는지 검증해야 합니다.  

## SendMoneyController 테스트 코드
다음은 SendMoneyController에서 sendMoney 메서드와 관련된 테스트 코드입니다.  

```java
@WebMvcTest(controllers = SendMoneyController.class)
class SendMoneyControllerTest {

	@Autowired
	private MockMvc mockMvc;

	@MockBean
	private SendMoneyUseCase sendMoneyUseCase;

	@Test
	void testSendMoney() throws Exception {

		mockMvc.perform(post("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
				41L, 42L, 500)
				.header("Content-Type", "application/json"))
				.andExpect(status().isOk());

		then(sendMoneyUseCase).should()
				.sendMoney(eq(new SendMoneyCommand(
						new AccountId(41L),
						new AccountId(42L),
						Money.of(500L))));
	}

}
```

## 스프링 부트 프레임워크를 활용한 통합 테스트 방법
SendMoneyController를 직접 생성하지 않고, MockMvc를 이용합니다.  

그리고 MockMvc에서 호출될 유스케이스의 가짜 객체를 생성하기 위해 MockBean 애너테이션을 사용합니다.  

이로 인해 유스케이스의 의존성 연결고리를 신경 안써도 되며 서비스의 호출, 결과를 임의로 조작하여 원하는  테스트를 할 수 있도록 지원합니다.  

mockMvc의 perform 메서드를 호출하여 sendMoney에서 보낼 메서드와 경로, 데이터 파라미터를 차례대로 매개변수로 넣어주고, Content-Type은 JSON 문자열 형태로 설정해줍니다.  

즉, 해당 조건으로 하여 목 HTTP 요청을 웹 컨트롤러에 보냅니다.  

그리고 나면 성공적으로 송금이 됬을 때 HTTP 응답 상태 코드인 200이 맞는지 검증하고, MockBean으로 모킹한 유스케이스가 잘 호출되었는지 검증합니다.  

MockMvc 객체를 이용했기 때문에 실제로 HTTP 프로토콜을 통해 테스트한 것은 아닙니다.  

다만 스프링 프레임 워크를 테스트할 필요는 없으니 프레임워크가 HTTP 프로토콜에 맞게 모든 것을 적절히 잘 변환한다고 믿는 것입니다.  

그리고 Mockito의 then 메서드를 통해 웹에서 해당 요청을 받았을 때 웹에서 전달 받은 데이터를 바탕으로 SendMoneyCommand를 생성하는 과정과 이 객체가 유스케이스로 전달되는지까지, 즉, 해당 데이터로 입력 모델을 만들어서 그 입력 모델을 매개변수로 하는 유스케이스의 메서드가 실제로 호출이 되는지까지 확인하고 있습니다.  

즉, JSON에서 SendMoneyCommand 객체로 매핑하고 있는 모든 과정을 웹 어댑터 테스트에서 다루고 있습니다.  

다시 말하자면 웹 어댑터 테스트에서는 웹 어댑터가 작동했을 때 이에 맞는 유스케이스가 실제로 호출되었는지 검증하고, 이 때 HTTP 응답이 기대한 상태를 반환했는지 이 2 가지를 검증합니다.  

## 웹 어댑터 테스트가 통합 테스트인 이유
WebMvcTest 애너테이션은 스프링이 특정 요청 경로, 자바와 JSON 간의 매핑, HTTP 입력 검증 등이 필요한 전체 객체 네트워크를 인스턴스화시켜서 테스트에서 웹 컨트롤러가 이 네트워크의 일부로서 잘 동작하는지 검증하도록 합니다.  

웹 컨트롤러가 스프링 프레임워크와 강하게 연관되어 있기 때문에 격리된 상태로 테스트하는 것 보다 스프링 프레임워크와 통합하여 테스트하는 것이 더 합리적입니다.  

웹 컨트롤러를 테스트할 때 스프링 프레임워크를 활용한 통합 테스트를 하지 않고, 단위 테스트로 테스트하면 모든 매핑, 유효성 검증, HTTP 항목에 대한 테스트를 일일이 다 할 수 없게 되어 프로덕션 환경에서 이러한 요소들이 정상적으로 작동할지 확신할 수 없게 됩니다.  

## SpringBootTest 애너테이션과 WebMvcTest 애너테이션의 차이
책에는 없지만 참고사항으로 SpringBootTest 애너테이션을 사용하면 스프링에 등록된 모든 빈을 다 등록하기 때문에 테스트 구동 시간이 오래 걸리고 테스트 단위가 커져서 디버깅이 어려워 질 수 있습니다.  

반면에 스프링에 등록된 모든 빈을 다 등록하지 않고, 웹 계층만 테스트 하고 싶을 때는 WebMvcTest 애너테이션을 사용하는 것이 더 좋습니다.  

이 때, 유스케이스 계층이나 영속성 계층에 대한 의존성 주입이 필요한 경우에는 아까 위에서 언급한 MockBean 애너테이션으로 의존성을 주입받아 테스트를 진행하면 됩니다.  

또한 WebMvcTest는 테스트하고 싶은 controller만 따로 설정할 수 있어서 SpringBootTest 애너테이션보다 테스트 속도가 빠르고, 테스트 단위도 좁힐 수 있습니다.  

따라서 책의 예시처럼 컨트롤러 계층만 빠르게 테스트하고 싶을 때는 WebMvcTest와 MockBean을 적절하게 사용하여 테스트하면 됩니다.  

# 통합 테스트로 영속성 어댑터 테스트하기
영속성 어댑터 테스트에서도 단순히 영속성 어댑터의 로직만 검증하고 싶은 것이 아니라 데이터베이스 매핑도 검증하고 싶기 때문에 단위 테스트보다는 통합 테스트를 적용하는 것이 더 바람직합니다.  

책에서는 Account 엔티티를 데이터베이스로부터 가져오는 메서드(application 패키지에 위치하는 아웃고잉포트인 LoadAccountPort 인터페이스의 loadAccount 메서드)와 새로운 계좌 활동을 데이터베이스에 저장하는 메서드(application 패키지에 위치하는 아웃고잉포트인 UpdateAccountStatePort 인터페이스의 updateActivities 메서드)까지 총 2개의 메서드를 테스트합니다.  

```java
@DataJpaTest
@Import({AccountPersistenceAdapter.class, AccountMapper.class})
class AccountPersistenceAdapterTest {

	@Autowired
	private AccountPersistenceAdapter adapterUnderTest;

	@Autowired
	private ActivityRepository activityRepository;

	@Test
	@Sql("AccountPersistenceAdapterTest.sql")
	void loadsAccount() {
		Account account = adapterUnderTest.loadAccount(new AccountId(1L), LocalDateTime.of(2018, 8, 10, 0, 0));

		assertThat(account.getActivityWindow().getActivities()).hasSize(2);
		assertThat(account.calculateBalance()).isEqualTo(Money.of(500));
	}

	@Test
	void updatesActivities() {
		Account account = defaultAccount()
				.withBaselineBalance(Money.of(555L))
				.withActivityWindow(new ActivityWindow(
						defaultActivity()
								.withId(null)
								.withMoney(Money.of(1L)).build()))
				.build();

		adapterUnderTest.updateActivities(account);

		assertThat(activityRepository.count()).isEqualTo(1);

		ActivityJpaEntity savedActivity = activityRepository.findAll().get(0);
		assertThat(savedActivity.getAmount()).isEqualTo(1L);
	}

}
```

## DataJpaTest 애너테이션
DataJpaTest 애너테이션은 스프링 데이터 리포지토리들을 포함해서 데이터베이스 접근에 필요한 객체 네트워크를 인스턴스화해야 한다고 스프링에 알려주는 역할을 합니다.  

## Import 애너테이션
Import 애너테이션을 추가해서 특정 객체가 이 네트워크 추가되었다는 것을 명확하게 표현해 줄 수 있습니다.  

Import에 명시된 AccountMapper는 영속성 어댑터를 테스트할 때 영속성 어댑터가 도메인 객체를 데이터베이스 객체로 매핑하는 등의 작업에 필요합니다.  

## loadsAccount 테스트 메서드
### Sql 애너테이션
Sql 애너테이션은 주로 테스트 클래스나 메서드에서 사용됩니다.  

Sql 애너테이션이 메서드 위에 붙어 있으면 테스트 메서드(책에서는 loadsAccount)를 실행하기 전에 SQL스크립트 혹은 쿼리를 먼저 실행시킵니다.  

Sql 애너테이션을 이용하면 현재 우리가 하려고 하는 통합 테스트에서 DB 스키마 생성과 초기 데이터 삽입 및 데이터 초기화 등을 손쉽게 할 수 있습니다.  

그래서 Sql 애너테이션은 대부분 테스트 독립성을 위해 데이터 초기화 용도로 사용됩니다.  

사용 방법은 간단합니다.  

사용할 SQL 스크립트 또는 쿼리 목록을 담은 파일을 테스트 코드의 recources 패키지 아래에 작성합니다.  

파일 명은 위 코드 에시처럼 마지막에 .sql을 붙이면 되는데 책의 코드에서는 AccountPersistenceAdapterTest.sql으로 파일명을 지었습니다.  

그럼 이제 파일을 만들었으니 db데이터 초기화를 원하는 테스트 클래스나 테스트 메서드에 Sql 애너테이션을 붙이고 괄호 안에 파일명을 적어줍니다.  

그러면 테스트 클래스나 테스트 메서드 전에 해당 파일에 있는 SQL 스크립트 또는 쿼리 목록이 실행됩니다.  

즉, Sql 애너테이션을 활용해 미리 SQL 스크립트를 만들어두고, 이를 이용해 데이터베이스를 원하는 특정 상태로 만듭니다.  

그런 다음 어댑터 API를 이용해 계좌를 가져온 후에 SQL 스크립트에서 설정한 상태값과 일치하는지 검증합니다.  

### updatesActivities 테스트 메서드
updatesActivities 테스트 메서드는 앞의 loadsAccount 테스트 메서드와 정반대로 작동합니다.  

새로운 AccountActivity를 가진 Account 객체를 만들어서 이를 영속성 계층, 즉, 데이터베이스에 저장하기 위해 영속성 어댑터로 전달합니다.  

그러고 나서 ActivityRepository의 API를 이용해 이 활동이 데이터베이스에 잘 저장이 됬는지 검증합니다.  

이 테스트에서는 **데이터베이스를 모킹하지 않았다는 점이 중요**합니다.  

테스트가 실제로 데이터베이스에 접근합니다.  

### 데이터베이스를 모킹하지 않는 이유
물론 데이터베이스를 모킹하더라도 테스트는 여전히 같은 코드 라인 수만큼 커버해서 똑같이 높은 커버리지를 보여줄 수 있습니다.  

하지만 데이터베이스를 모킹한 테스트는 실제로 데이터베이스와 연동했을 때 발생할 수 있는 SQL 구문 오류나 데이터베이스 테이블과 자바 객체 간의 매핑 에러 등을 전부 커버할 수는 없습니다.  

스프링에서는 기본적으로 아무것도 설정할 필요가 없는 인메모리(in-memory) 데이터베이스를 테스트에서 사용하여 매우 편리합니다.  

하지만 프로덕션 환경에서는 인메모리 데이터베이스를 사용하는 경우는 거의 없기 때문에 인메모리 데이터베이스에서 테스트를 완벽히 통과하더라도 실제 데이터베이스에서는 문제가 발생할 확률이 높습니다.  

예를 들어 데이터베이스마다 고유한 SQL 문법의 차이로 인한 문제 발생을 테스트에서 인메모리 데이터베이스를 사용할 경우 발견할 수 없습니다.  

따라서 영속성 어댑터 테스트는 실제 데이터베이스를 대상으로 진행해야 합니다.  

만약에 테스트에서 인메모리 데이터베이스를 사용하면 특정 방식으로 테이터베이스를 설정하거나 데이터베이스별로 두 가지 버전의 데이터베이스 마이그레이션 스크립트를 만들어야 하는데 실제 데이터베이스를 대상으로 테스트를 하면 이러한 것들을 할 필요가 없어집니다.  

### Transactional 애너테이션
참고로 이는 책에 있는 내용은 아니지만 영속성 계층을 실제 데이터베이스 기반에서 테스트하다보면 테스트한 데이터가 db에 저장될 수 있고, 어떠한 경우에는 이것이 싫을 수도 있고, 대다수의 경우 이것이 문제가 될 수 있습니다.  

테스트는 아무리 실행해도 그 결과가 같아야 하고(1번 통과한 테스트는 100번을 실행하더라도 통과해야하고), 테스트 메서드의 순서에 영향을 받지 않아야 하는데(테스트 클래스에 여러 메서드가 있고 이를 한 번에 다 실행시키면 그 순서는 장담할 수 없음) 테스트 메서드가 누가 먼저 실행되냐에 따라 데이터베이스에 다른 정보가 저장되어 에러가 발생할 수도 있습니다.(예를 들어, 중복이 없어야 하는데 다른 메서드가 먼저 실행되어 데이터베이스에 데이터 중복이 발생할 수도 있습니다.)  

테스트 클래스나 테스트 메서드 위에 Transactional 애너테이션을 붙이면 이러한 상황이 막을 수 있습니다.  

실제 프로덕션 코드와 다르게 테스트 코드에 Transactional 애너테이션이 있으면 테스트가 끝나고 자동으로 rollback을 하기 때문에(에러 발생 여부와 상관없이) 테스트에서 실행한 내용들은 데이터베이스에 저장되지 않습니다.  

# 시스템 테스트로 주요 경로 테스트하기
테스트 피라미드의 최상단에 있는 시스템 테스트는 전체 애플리케이션을 띄우고 API를 통해 요청을 보내고, 아키텍처의 모든 계층이 조화롭게 잘 작동하는지 검증합니다.  

'송금하기' 유스케이스의 시스템 테스트에서는 HTTP로 부터 받은 요청을 애플리케이션에 보내고, 애플리케이션이 계좌의 잔고를 확인하여 다른 계좌로 송금한 후 정상적으로 처리된 것에 대한 HTTP의 응답 및 데이터를 검증합니다.  

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class SendMoneySystemTest {

	@Autowired
	private TestRestTemplate restTemplate;

	@Autowired
	private LoadAccountPort loadAccountPort;

	@Test
	@Sql("SendMoneySystemTest.sql")
	void sendMoney() {

		Money initialSourceBalance = sourceAccount().calculateBalance();
		Money initialTargetBalance = targetAccount().calculateBalance();

		ResponseEntity response = whenSendMoney(
				sourceAccountId(),
				targetAccountId(),
				transferredAmount());

		then(response.getStatusCode())
				.isEqualTo(HttpStatus.OK);

		then(sourceAccount().calculateBalance())
				.isEqualTo(initialSourceBalance.minus(transferredAmount()));

		then(targetAccount().calculateBalance())
				.isEqualTo(initialTargetBalance.plus(transferredAmount()));

	}

	private Account sourceAccount() {
		return loadAccount(sourceAccountId());
	}

	private Account targetAccount() {
		return loadAccount(targetAccountId());
	}

	private Account loadAccount(AccountId accountId) {
		return loadAccountPort.loadAccount(
				accountId,
				LocalDateTime.now());
	}


	private ResponseEntity whenSendMoney(
			AccountId sourceAccountId,
			AccountId targetAccountId,
			Money amount) {
		HttpHeaders headers = new HttpHeaders();
		headers.add("Content-Type", "application/json");
		HttpEntity<Void> request = new HttpEntity<>(null, headers);

		return restTemplate.exchange(
				"/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
				HttpMethod.POST,
				request,
				Object.class,
				sourceAccountId.getValue(),
				targetAccountId.getValue(),
				amount.getAmount());
	}

	private Money transferredAmount() {
		return Money.of(500L);
	}

	private Money balanceOf(AccountId accountId) {
		Account account = loadAccountPort.loadAccount(accountId, LocalDateTime.now());
		return account.calculateBalance();
	}

	private AccountId sourceAccountId() {
		return new AccountId(1L);
	}

	private AccountId targetAccountId() {
		return new AccountId(2L);
	}

}
```

## SpringBootTest 애너테이션
아까 위에서 언급한 것처럼 SpringBootTest 애너테이션은 스프링이 애플리케이션을 구성하는 모든 객체 네트워크를 뛰우게 함으로써 모든 계층의 조화를 테스트해야하는 시스템 테스트에 사용하기 적합합니다.  

참고로 SpringBootTest 애너테이션은 랜덤 포트로 이 애플리케이션을 띄우도록 설정합니다.  

## TestRestTemplate 클래스
TestRestTemplate은 REST(Representational State Transfer) API의 Test를 최적화 하기 위해 만들어진 클래스입니다.  

HTTP 요청 후 데이터를 응답(response) 받을 수 있는 템플릿 객체입니다.  

책에서는 whenSendMoney 메서드 내부에서 TestRestTemplate의 exchange 메서드를 사용하고 있습니다.  

이 exchange 메소드는 주로 update할 때 사용됩니다.  

반환 값이 ResponseEntity이고, Http header를 변경할 수 있습니다.  

책의 테스트 코드에서 exchange의 매개변수로 경로와 그 경로의 변수에 맞는 파라미터값, post 메서드 등을 넣었는데 즉, 해당 경로와 파라미터값을 넘기면서 post로 요청했을 때 그 응답(ResponseEntity)을 return합니다.

## ResponseEntity 클래스

ResponseEntity 클래스는 스프링이 제공하는 HttpEntity 클래스(HttpEntity 클래스는 Http request/response가 이루어 질 때 Http header와 body를 포함)를 상속받습니다.  

따라서 ResponseEntity 클래스는 부모인 HttpEntity 클래스의 기능을 사용할 수 있는데, 사용자의 HttpRequest에 대한 응답하는 데이터(Http Status Code, Header, Body 등)를 가지고 있습니다.  

## TestRestTemplate와 MockMvc의 차이
책에서 통합 테스트를 할 때는 MockMvc를 썼는데 시스템 테스트를 할 때는 TestRestTemplate를 사용하고 있습니다.  

이는 통합 테스트를 할 때는 WebMvcTest 애너테이션을 사용하다가 시스템 테스트를 할 떄는 SpringBootTest 애너테이션 사용하는 것과도 연관이 있습니다.  

WebMvcTest 애너테이션과 SpringBootTest 애너테이션의 차이는 위에서 설명하였고, **TestRestTemplate와 MockMvc의 차이점은 Servlet Container를 사용하느냐 안 하느냐의 차이**입니다.

(Servlet Container는 웹서버에서 Clinet의 HTTP Request를 받고 그에 대해 HTTP Response를 할 수 있게, 웹 서버와 소켓을 만들어 통신해주는 역할을 하며, 대표적으로 무료 서비스인 Tomcat(톰캣)이 여기에 해당합니다.)

MockMvc는 Servlet Container를 생성하지 않는 반면에 TestRestTemplate은 Servlet Container를 사용합니다.  

따라서 TestRestTemplate을 이용하여 테스트하면 Servlet Container를 사용하기 때문에 마치 실제 서버가 동작하는 것과 유사한 환경에서 테스트를 수행할 수 있습니다.  

그래서 시스템 테스트에 TestRestTemplate을 이용하는 것이 더 바람직합니다.  

요약해보자면 통합 테스트를 할 때는 Servlet Container를 생성하지 않는 환경의 MockMvc와 WebMvcTest 애너테이션을 사용하여 시스템 테스트보다는 상대적으로 가볍고 좀 더 집중하여 테스트합니다.  

시스템 테스트를 할 때는 Servlet Container를 사용하는 TestRestTemplate와 SpringBootTest 애너테이션을 이용해 실제 서버가 동작하는 것과 유사한 환경에서 종합적으로 테스트 할 수 있습니다.  

대신 통합 테스트를 할 때보다 무겁기 때문에 구동 시간이 오래 걸리고 포괄적으로 테스트하며 실제 프로덕션 환경에 더 가깝게 테스트하기 위해 실제로(mockMvc의 경우 가짜이지만) HTTP 통신을 합니다..  

실제 HTTP와 통신을 하기 때문에 실제 출력 어댑터도 이용하는데 책에서 출력 어댑터는 애플리케이션과 데이터베이스를 연결하는 영속성 어댑터입니다.  

상황에 따라서는 출력 어댑터가 다양하기 때문에 모든 출력 어댑터를 실제로 다 이용할 수는 없고, 모킹을 해야할 필요가 생길 수 있습니다.  

하지만 육각형 아키텍처를 이용하면 몇 개의 출력 포트 인터페이스만 모킹하면 되기 때문에 이 문제를 쉽게 해결할 수 있습니다.  

## 헬퍼 메서드(도메인 특화 언어)
테스트의 가독성을 높이기 위해 테스트 코드에 모든 코드를 다 넣지 않고, 의미적으로 덩어리로 합칠 수 있는 부분들을 헬퍼 메서드로 만들어 따로 밖으로 빼서 테스트 메서드에서 필요할 때 호출해줍니다.(테스트 메서드에서만 호출될 것이기 때문에 private으로 설정함)

이러한 헬퍼 메서드들은 테스트에서 여러 가지 상태를 검증할 때 사용할 수 있는 도메인 특화 언어라고 할 수 있습니다.  

시스템 테스트는 단위 테스트나 통합 테스트보다 훨씬 더 실제 사용자를 가정하여 테스트할 수 있기 때문에 사용자의 관점에서 애플리케이션을 검증할 수 있습니다.  

따라서 시스템 테스트를 할 때 도메인 특화 언어를 잘 설정해놓으면(헬퍼메서드 명을 잘지으면) 프로그래머는 아니지만 도메인 전문가인 사람들도 태스트에 대해 생각하고 피드백을 줄 수 있습니다.  

## sendMoney 테스트 메서드
sendMoney 테스트 메서드를 실행하기 전에 먼저 Sql 애너테이션을 통해 테스트 resources패키지에 위치한 "SendMoneySystemTest.sql"에 접근하여 데이터베이스를 원하는 특정 상태로 초기화시킵니다.  

이 후 출금하려는 계좌의 잔액(intitialSourceBalance)을 구하고, 송금받으려는 계좌의 잔액(initialTargetBalance)을 먼저 구합니다.  

그리고 아까 위에서 설명한 헬퍼 메서드인 whenSendMoney 메서드를 호출하여 응답(response)을 반환 받습니다.  

먼저 이 응답이 원하는 상태인지(HttpStatus.OK) 확인합니다.  

이후 whenSendMoney 메서드 호출 이후(즉, 송금이 완료된 후) 출금된 계좌에서 다시 구한 잔액이 초기(출금 전) 잔액(intitialSourceBalance)에서 보낸 금액(transferredAmount)을 차감한 금액과 같은지 검증합니다.  

또한  whenSendMoney 메서드 호출 이후(즉, 송금이 완료된 후) 송금 받은 계좌에서 다시 구한 잔액이 초기(송금 받기 전) 잔액(initialTargetBalance)에서 받은 금액(transferredAmount)을 더한 금액과 같은지 검증합니다.  

이로써 시스템 테스트가 끝이 납니다.  

## 시스템 테스트를 해야하는 이유 
시스템 테스트를 하면서 앞에서 만든 단위 테스트, 그리고 통합 테스트와 중복하여 테스트 하는 부분이 많을 것입니다.  

그럼에도 불구하고 시스템 테스트를 해야하는 이유는 단위 테스트와 통합 테스트에서 미처 발견하지 못하는 다른 종류의 버그를 시스템 테스트를 통해 발견할 수 있기 때문입니다.  

예를 들어 단위 테스트나 통합 테스트를 할 때는 알지 못했을 계층 간 매핑 버그를 들 수 있습니다.  

시스템 테스트는 여러 개의 유스케이스를 결합해서 시나리오를 만들 때 훨씬 더 좋습니다.  

각각의 시나리오는 사용자가 애플리케이션을 사용하면서 거쳐갈 특정 경로, 즉, 사용자가 어떻게 해당 프로그램을 사용할 지에 대한 모의 상황을 가정하는 것입니다.  

이렇게 시스템 테스트를 통해 메인 시나리오들이 커버된다면 최신 변경사항들로 인해 애플리케이션에 에러가 발생하지 않음을 확신하며 배포할 수 있습니다.  

# 얼마만큼의 테스트가 충분할까?
테스트를 실행했을 때 실행된 라인의 수를 전체 라인 수 대비 퍼센티지로 나타낸 값인 라인 커버리지는 테스트 성공을 측정하는데 있어서 잘못된 지표입니다.  

그 이유는 코드의 중요한 부분이 커버가 되지 않아도 수치는 높게 나올 수 있고, 그렇다면 이는 나중에 심각한 오류가 발생할 수 있기 때문입니다.  

그래서 100퍼센트를 제외한 어떤 목표도 완전히 무의미하며 심지어 100퍼센트라 하더라도 모든 버그를 다 잡았다고 확신할 수 없습니다.  

그래서 이보다는 얼마나 마음 편하게 소프트웨어를 배포할 수 있느냐를 테스트의 성공 기준으로 삼는 것이 좋습니다.  

즉, 테스트를 실행한 후에 소프트웨어를 배포해도 될 만큼 테스트 결과를 신뢰한다면 충분합니다.  

또한 더 자주 배포할수록 테스트를 더 자주하기 때문에 테스트 결과를 더욱 신뢰할 수 있습니다.  

애플리케이션을 배포하는 초창기에는 아무래도 테스트에 대한 신뢰가 약하고, 이 상태로 프로그램을 배포할 수 밖에 없지만(처음부터 완벽할 수 없으니) 배포 후 발생하는 프로덕션의 버그를 수정하고 이로부터 배우는 것으 목표로 삼는다면 괜찮습니다.  

배포 후 발생한 버그에 대해서 내 테스트코드가 이 버그를 잡지 못한 이유를 생각하고 이에 대한 답변을 기록하고 이 버그를 커버할 수 있는 테스트를 추가해야 합니다.  

이러한 방식을 고수하면 시간이 흘러감에 따라 남겨둔 기록들이 상황이 개선되고 있다는 것을 증명해줄 것이고, 배포할 때 마음이 편해질 것입니다.  

## 육각형 아키텍처에서 사용하는 테스트 전략
<ul>
	<li>도메인 엔티티를 구현할 때는 단위 테스트로 커버하자</li>
	<li>유스케이스를 구현할 때는 단위 테스트로 커버하자</li>
	<li>어댑터를 구현할 때는 통합 테스트로 커버하자</li>
	<li>사용자가 취할 수 있는 중요 애플리케이션 경로는 시스템 테스트로 커버하자</li>
</ul>

**구현할 때는**이라는 문구를 명심해야 합니다.  

만약에 테스트가 기능 개발 후에 이뤄진다면 하기 싫은 귀찮은 작업이 될 확률이 높습니다.  

왜냐하면 '이미 구현이 다 되어서 잘돌아가는데 굳이 이걸 또 해야하나'라는 생각이 들 수 있기 때문입니다.  

하지만 개발하는 중에 테스트를 한다면 테스트는 더이상 사후에 하는 귀찮은 작업이 아니라 중요한 개발 도구라고 생각하게 될 것입니다.  

그래서 **구현하는 중에** 테스트틀 적절히 사용하여 기능을 개발해야 합니다.  

하지만 새로운 필드를 추가할 때마다 테스트를 고치는 데 한 시간을 써야 한다면 뭔가 잘못된 길로 가고 있는 것입니다.  

이는 테스트가 코드의 구조적 변경에 너무 취약할 확률이 높기 때문에 어떻게 이를 개선할 수 있을지 고민해야 합니다.  

리팩터링할 때 마다 테스트 코드도 변경해야 한다면 테스트는 다시 귀찮은 작업으로 전락할 확률이 높습니다.  

# 유지보수 가능한 소프트웨어를 만드는 데 어떻게 도움이 될까?
육각형 아키텍처는 도메인 로직과 바깥으로 향하는 어댑터를 깔끔하게 분리합니다.  

이로 인해 핵심 도메인 로직은 단위 테스트로, 어댑터는 통합 테스트로 처리하는 명확한 테스트 전략을 수행할 수 있습니다.  

입출력 포트는 테스트에서 아주 뚜렷한 모킹 지점이 됩니다.  

각 포트에 대해 모킹할지, 실제 구현을 이용할 지 선택할 수 있습니다.  

만약 포트가 아주 작고 핵심만 담고 있다면(포트 인터페이스가 더 적은 메소드를 제공할수록) 헷갈리지 않고 쉽게 모킹할 수 있습니다.  

모킹하는 것이 힘들거나 코드의 특정 부분을 커버하기 위해 어떤 종류의 테스트를 써야 할지 모르겠다면 이는 경고 신호입니다.  

즉, 테스트는 아키텍처의 문제에 대해 경고하고 유지보수 가능한 코드를 만들기 위해 올바른 길을 인도하는 중요한 역할을 합니다.

책에서는 이를 카나리아에 비교하고 있길래 카라리아가 무엇인지 검색했습니다.  

카나리아를 새의 이름이라서 처음에 의아했습니다.  

옛날에는 광산 안에서 산소분압 측정이나 군대에서 난방을 할 때 일산화탄소 측정 역할을 하기 위해 카나리아를 길렀다고 합니다.  

그래서 카나리아의 반응을 보고 광산이 무너지거나 일산화탄소 중독이 되기 전에 미리 알 수 있었다고 합니다.  

그래서 영단어 Canary(카나리아)는 새의 이름도 맞지만 또다른 의미로 위험이나 문제에 대한 경고 및 예방을 위한 장치나 테스트 버전을 뜻하기도 합니다.  

그래서 저자는 테스트 코드의 역할이 카나리아와 같다고 이야기 하고 있습니다.  

# 마치며

여기까지 7장에 대한 정리를 마쳤습니다.  

테스트 코드에 대해서 잘 몰라서 개인적으로 조사도 하면서 많은 시간을 할애하여 읽었습니다.  

그래서 블로그에 올린 내용 중에는 책에는 없지만 제가 이해하기 위해 찾아 본 내용도 꽤 있습니다.  

긴 글 읽어주셔서 감사합니다^^
