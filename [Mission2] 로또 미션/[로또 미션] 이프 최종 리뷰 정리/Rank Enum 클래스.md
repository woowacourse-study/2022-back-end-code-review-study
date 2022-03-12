## Rank Enum 클래스

### 기존의 코드
```java
public enum Rank {

    FIFTH_GRADE(3, false, 5_000L),
    FOURTH_GRADE(4, false, 50_000L),
    THIRD_GRADE(5, false, 1_500_000L),
    SECOND_GRADE(5, true, 30_000_000L),
    FIRST_GRADE(6, false, 2_000_000_000L);

    private final int matchCount;
    private final boolean bonusMatched;
    private final long prizeMoney;

    Rank(final int matchCount, final boolean bonusMatched, final long prizeMoney) {
        this.matchCount = matchCount;
        this.bonusMatched = bonusMatched;
        this.prizeMoney = prizeMoney;
    }

    public static Optional<Rank> of(final int matchCount, final boolean bonusMatched) {
        return Arrays.stream(Rank.values())
                .filter(rank -> rank.matches(matchCount, bonusMatched))
                .findFirst();
    }

    private boolean matches(final int matchCount, final boolean bonusMatched) {
        return (matchCount == this.matchCount) && (bonusMatched == this.bonusMatched);
    }

   ...

}
```

일치 개수 상 1등/4등/5등이어야 하지만 보너스 볼을 포함하고 있으면 당첨으로 인정되지 않는 문제가 존재했다.

2등과 3등의 판별 과정에서는 보너스 볼 포함 여부가 사용되므로 true/false 값을 지니고 있어야 하는데,  
1등,4등,5등은 보너스 볼 일치 여부가 필요없음에도 false로 값이 지정되었기 때문에 이러한 문제가 발생했다.

그래서 고민해본 결과, 보너스 볼 일치 여부(bonusMatched) 외에
보너스 볼 필요 여부(bonusRequired)를 이용하기로 결정했다.

### Rank 1차 수정 코드

```java
public enum Rank {

    FIFTH_GRADE(3, 5_000L),
    FOURTH_GRADE(4, 50_000L),
    THIRD_GRADE(5, false, 1_500_000L),
    SECOND_GRADE(5, true, 30_000_000L),
    FIRST_GRADE(6, 2_000_000_000L);

    private final int matchCount;
    private final boolean bonusRequired;
    private final boolean bonusMatched;
    private final long prizeMoney;

    Rank(final int matchCount, final boolean bonusRequired, final boolean bonusMatched, final long prizeMoney) {
        this.matchCount = matchCount;
        this.bonusRequired = bonusRequired;
        this.bonusMatched = bonusMatched;
        this.prizeMoney = prizeMoney;
    }

    Rank(final int matchCount, final boolean bonusMatched, final long prizeMoney) {
        this(matchCount, true, bonusMatched, prizeMoney);
    }

    Rank(final int matchCount, final long prizeMoney) {
        this(matchCount, false, false, prizeMoney);
    }

    public static Optional<Rank> of(final int matchCount, final boolean bonusMatched) {
        return Arrays.stream(Rank.values())
                .filter(rank -> rank.matches(matchCount, bonusMatched))
                .findAny();
    }

    private boolean matches(final int matchCount, final boolean bonusMatched) {
        return matchesCount(matchCount) && matchesBonus(bonusMatched);
    }

    private boolean matchesCount(final int matchCount) {
        return matchCount == this.matchCount;
    }

    private boolean matchesBonus(final boolean bonusMatched) {
        if (this.bonusRequired) {
            return bonusMatched == this.bonusMatched;
        }
        return true;
    }
    
    ...
}
```

위의 코드를 봐도 알 수 있듯.. 많이 더럽다ㅋㅋㅋㅋ이게 아닌데ㅜ

위와 같은 문제가 나온 이유는, `보너스 볼 포함 여부`가 `true`/`false`로 분리되는 것이 아니라  
1,4,5등과 같이 `포함 여부`가 상관없는 경우가 존재했기 때뭄에  
`보너스 볼 판별 필요 여부`를 추가로 생성하다보니 코드가 어지럽혀진 것이다.

### Rank 최종 코드

```java
public enum Rank {

    FIFTH_GRADE(BonusMatched.CAN_BE_WHATEVER, 3, 5_000L),
    FOURTH_GRADE(BonusMatched.CAN_BE_WHATEVER, 4, 50_000L),
    THIRD_GRADE(BonusMatched.MUST_BE_FALSE, 5, 1_500_000L),
    SECOND_GRADE(BonusMatched.MUST_BE_TRUE, 5, 30_000_000L),
    FIRST_GRADE(BonusMatched.CAN_BE_WHATEVER, 6, 2_000_000_000L);

    private final BonusMatched bonusMatched;
    private final int matchCount;
    private final long prizeMoney;

    private enum BonusMatched {

        CAN_BE_WHATEVER(true, false),
        MUST_BE_FALSE(false),
        MUST_BE_TRUE(true);

        private final List<Boolean> states;

        BonusMatched(final boolean state1, final boolean state2) {
            this.states = List.of(state1, state2);
        }

        BonusMatched(final boolean state) {
            this.states = List.of(state);
        }

        private boolean matches(final boolean bonusMatched) {
            return states.stream()
                    .anyMatch(state -> state == bonusMatched);
        }

    }

    Rank(final BonusMatched bonusMatched, final int matchCount, final long prizeMoney) {
        this.bonusMatched = bonusMatched;
        this.matchCount = matchCount;
        this.prizeMoney = prizeMoney;
    }

    public static Optional<Rank> of(final int matchCount, final boolean bonusMatched) {
        return Arrays.stream(values())
                .filter(rank -> rank.matchCount == matchCount)
                .filter(rank -> rank.bonusMatched.matches(bonusMatched))
                .findAny();
    }

    public boolean isTrueThatBonusMatchMustBeTrue() {
        return bonusMatched.equals(BonusMatched.MUST_BE_TRUE);
    }

    ...
}
```

최종적으로 Enum 내부 클래스를 통해 문제를 해결했다. 코드가 깔끔해진 것을 보니 뿌듯하다.