## Enum 클래스를 통한 상수 문자열 관리

### 수정 전의 코드
```java
public class LottoException extends IllegalArgumentException {

    private static final String ERROR_MESSAGE_PREFIX = "[ERROR] ";

    public LottoException(LottoExceptionStatus lottoExceptionStatus) {
        super(ERROR_MESSAGE_PREFIX + lottoExceptionStatus.getMessage());
    }

}
```
```java
public interface LottoExceptionStatus {

    String getMessage();

}
```

```java
public enum MoneyExceptionStatus implements LottoExceptionStatus {

    MONEY_MUST_BE_NUMERIC("구입 금액은 숫자여야 합니다."),
    MONEY_CANNOT_BE_ZERO("구입 금액은 0원이 될 수 없습니다."),
    MONEY_MUST_BE_DIVISIBLE("구입 금액은 1000원으로 나누어 떨어져야 합니다.");

    private final String message;

    MoneyExceptionStatus(final String message) {
        this.message = message;
    }

    public String getMessage() {
        return this.message;
    }

}
```

> 이렇게 ExceptionStatus라는 개념을 도입하여 메시지를 enum으로 관리하는 것에는 어떤 장점이 있을까요?

이와 같이 하는 경우, LottoExceptionStatus에 미리 정해져있는 예외메시지 이외의 값을  
LottoException 에 담는 것을 차단할 수 있는 장점이 있습니다.  

개발자는 throw new LottoException("이거 예외인가요?")와 같이 함부로 String 값을  
담아서 예외를 던질 수 없고, LottoExceptionStatus를 implement한 enum 클래스를 통해  
예외메시지를 관리해야만 합니다.  

때문에 ExceptionStatus 개념의 도입은 예외메시지를 별도 관리한다는 점에서 유지보수가 편리해지며  
사전에 지정된 예외메시지 외의 값을 임의로 사용하지 못하도록 강제하기 때문에  
프로그래밍 관점에서 안전성이 보장됩니다.  

예외에 함부로 String 값을 담으면 안 되는 걸까요? 만약 프로그램이 커진다면 예외도 많아질텐데, 새로운 예외가 생길 때마다 LottoExceptionStatus의 구현체를 만들어야 하는 것이 부담이 되지는 않을까요? 예외 메시지를 별도 관리한다는 점에서 유지보수가 편리해진다고 하셨는데, 지금 LottoExceptionStatus의 구현체가 모두 흩어져 있기 때문에 원하는 예외 메시지를 담은 구현체가 이미 있는지 찾기도 쉽지 않아 보여요.  

꼭 제 의견이 정답이라기보다는, 다른 사람은 이렇게 생각할 수도 있다는 정도로 봐주셔도 될 것 같아요.  

> 예외에 함부로 String 값을 담으면 안 되는 걸까요? 만약 프로그램이 커진다면 예외도 많아질텐데, 새로운 예외가 생길 때마다 LottoExceptionStatus의 구현체를 만들어야 하는 것이 부담이 되지는 않을까요? 예외 메시지를 별도 관리한다는 점에서 유지보수가 편리해진다고 하셨는데, 지금 LottoExceptionStatus의 구현체가 모두 흩어져 있기 때문에 원하는 예외 메시지를 담은 구현체가 이미 있는지 찾기도 쉽지 않아 보여요.  
> 꼭 제 의견이 정답이라기보다는, 다른 사람은 이렇게 생각할 수도 있다는 정도로 봐주셔도 될 것 같아요.  

좋은 말씀 감사합니다ㅜ!! 이 부분에 대한 제 생각은 처음과 달리 지금은 달라졌는데요.  
우선 예외메시지를 Enum으로 관리하고 `LottoException`의 생성자 매개변수 타입이 `LottoExceptionStatus`인 것에 대해서는 변함이 없으나, `LottoExceptionStatus`의 구현체로 관리하는 것에 대해서는 지적하신 부분을 수용하였습니다.  

하나씩 말씀드리자면, 우선 `LottoExceptionStatus`라는 Enum 클래스를 통해 예외메시지를 다루게 된 이유에 대해 설명드리겠습니다. 예외메시지를 Enum으로 관리하는 것은 관리측면에서도 확장측면에서도 큰 장점이 있다고 생각합니다. 관리측면에서 바라봤을 때, 특정 예외메시지를 수정하기 위해 프로젝트 전체를 확인할 필요 없이 예외메시지가 모여있는 `LottoExceptionStatus` 클래스만 확인하고 수정하면 되므로 관리하기 용이하다는 장점이 있습니다. 그리고 `LottoExceptionStatus`를 Enum 클래스로 생성한 이유는 확장성 때문인데, 에러코드와 같이 예외메시지와 연관된 추가적인 필드값을 추가/관리해야 하는 상황에서 코드 변경에 사용될 비용을 고려한다면 처음부터 Enum 클래스로 관리하는 것이 오히려 더 유리하다고 판단했습니다.  

`LottoException`의 생성자 매개변수 타입으로 `LottoExceptionStatus`로 제한한 이유는 `LottoExceptionStatus`의 목적 때문인데요. 앞서 서술하였듯 `LottoExceptionStatus`은 예외메시지를 한 곳에서 관리하려는 목적을 지니고 있습니다. 예외메시지를 한 곳에서 관리하기 위해 `LottoExceptionStatus` Enum 클래스를 생성한 것인데, `LottoExceptionStatus`에 예외메시지를 정의해놓지 않고서, 함부로 `LottoExcpetion`을 생성하고 사용하는 것은 `LottoExceptionStatus`의 존재 의의에 어긋난다고 생각했습니다. 따라서 `LottoException`의 생성자 매개변수를 String 타입이 아닌 `LottoExceptionStatus`로 제한함으로써 예외메시지는 해당 클래스에서 관리되어야 함을 보장했습니다.  

그에 반해 `LottoExceptionStatus` 구현체 활용에 대해서는 이번 미션2를 진행하면서 생각이 바뀌었는데요. 지금은 `LottoExceptionStatus` Enum 클래스 안에 모든 예외메시지를 담아둔 것에 비해, 기존에는  `LottoExceptionStatus`를  interface로 두고서 `MoneyExceptionStatus` 등의 Enum 구현체를 생성하여 예외메시지를 도메인별로 묶었습니다. 당시에는 연관된 메시지끼리 묶어서 분리했기 때문에 유지보수가 좋다고 생각했지만, 코니의 피드백을 보고서 `예외메시지를 한곳에 모아 놓은 이유가 관리를 용이하게 하기 위함인데 굳이 도메인별로 쪼갤 필요가 있을까` 하는 의문을 갖게 되었습니다. 오히려 `로또 수동 구매 기능 구현 과정`에서는 `이 예외메시지는 어느 구현체에 넣어야하는거지?` 라는 생각을 가질 정도로 저 스스로 구현체 활용에 대해 혼란을 느끼기도 했습니다. 그래서 결국 도메인별로 분리한 인터페이스 구현체를 하나로 합쳐서 지금의 `LottoExceptionStatus`를 이용하게 되었습니다.  

> 이렇게 직접 느껴야만 배울 수 있는 것들이 많더라구요 👍 저도 공감이 많이 됩니다.

<br>

### 수정 후의 코드
```java
public class LottoException extends IllegalArgumentException {

    private static final String ERROR_MESSAGE_PREFIX = "[ERROR] ";

    public LottoException(LottoExceptionStatus lottoExceptionStatus) {
        super(ERROR_MESSAGE_PREFIX + lottoExceptionStatus.getMessage());
    }

}
```
```java
public enum LottoExceptionStatus {
 
    READER_CANNOT_READ("입력을 읽어들일 수 없습니다."),

    MONEY_MUST_BE_NUMERIC("구입 금액은 숫자여야 합니다."),
    MONEY_CANNOT_BE_ZERO("구입 금액은 0원이 될 수 없습니다."),
    MONEY_MUST_BE_DIVISIBLE("구입 금액은 1000원으로 나누어 떨어져야 합니다."),

    TICKET_COUNT_MUST_BE_NUMERIC("로또 개수는 숫자여야 합니다."),

    TICKET_NUMBERS_CANNOT_BE_OUT_OF_SIZE("로또 번호는 6개로 구성되어야 합니다."),
    TICKET_NUMBERS_CANNOT_BE_DUPLICATED("로또 번호는 중복될 수 없습니다."),

    BALL_NUMBER_MUST_BE_NUMERIC("번호는 숫자여야 합니다."),
    BALL_NUMBER_CANNOT_BE_OUT_OF_RANGE("번호의 범위는 1부터 45까지여야 합니다.");

    private final String message;

    LottoExceptionStatus(final String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

}
```