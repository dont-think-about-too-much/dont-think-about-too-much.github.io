> ["TDD 리팩토링 by 자바지기 박재성님"](https://www.youtube.com/watch?v=bIeqAlmNRrA&t=552s&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)의 강연을 보고 리팩토링부분만 정리한 글이다.

# 리팩토링 연습 - 메서드 분리

오늘 다룰 메서드이다.

값이 null이거나 빈문자열일 때는 0을 반환하고 쉽표와 콜론dml 경우엔 split을 해서 for문돌면서 Int로 바꾸서 합을 구한다.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        int result = 0;
        if (text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            for (String value : values) {
                result += Integer.parseInt(value);
            }
        }
        return result;
    }
}
```

잠시 5분동안 위 코드를 스스로 리팩토링해보자. 여기서 더 뭘 줄이냐 싶을텐데. 할게 꽤 있다.
(내 경우엔 if 조건문을 메서드로 만드는 것밖에 생각못했다.)

5분동안만 해보자.
<br><br><br><br>

5분이 지났으면 같이 리팩토링해보자.

# 한 메서드에 오직 한 단계의 들여쓰기(indent)만 한다.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        int result = 0;
        if (text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            for (String value : values) {               //
                result += Integer.parseInt(value);      //  여기가 Indent가 2다.
            }                                           //
        }
        return result;
    }
}
```

==> **메서드를 분리하여 인덴트를 줄인다.**

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        int result = 0;
        if (text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            result = sum(values);       // sum 함수를 만들어서 인덴트를 줄였다.
        }                               // Indent 가 1이 됐다.
        return result;
    }

    private static int sum(String[] values) {
        int result = 0;
        for (String value : values) {
            result += Integer.parseInt(value);
        }
        return result;
    }
}
```

# else 예약어를 쓰지 않는다.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        int result = 0;                             //
        if (text == null || text.isEmpty()) {       //
            result = 0;                             //
        } else {                                    // else 를 쓰지 않는다.
            String[] values = text.split(",|:");    //
            result = sum(values);                   //
        }                                           //
        return result;
    }

    private static int sum(String[] values) {
        int result = 0;
        for (String value : values) {
            result += Integer.parseInt(value);
        }
        return result;
    }
}
```

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        if (text == null || text.isEmpty()) {       //
            result = 0;                             // 불필요한 로컬변수를 만들
        }                                           // 필요가 없어진다 !
        String[] values = text.split(",|:");        //
        return sum(values);                         // else 를 쓰지 않으면
    }                                               // indent 또한 줄어든다.

    private static int sum(String[] values) {
        int result = 0;
        for (String value : values) {
            result += Integer.parseInt(value);
        }
        return result;
    }
}
```

# 메서드가 한가지 일만 하도록 구현하기

sum 함수가 계속 거슬린다. 하나의 일만하는거 같지가 않다.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        if (text == null || text.isEmpty()) {
            result = 0;
        }
        String[] values = text.split(",|:");
        return sum(values);
    }

    private static int sum(String[] values) {   //
        int result = 0;                         //
        for (String value : values) {           // sum 메서드가
            result += Integer.parseInt(value);  // string을 int로 변환도 하고
        }                                       // 더하기까지 한다.
        return result;                          //
    }
}
```

sum 메서드가 인트 변환뿐만아니라 더하기까지 해버린다. 두개의 메서드로 나누자.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        if (text == null || text.isEmpty()) {
            result = 0;
        }
        String[] values = text.split(",|:");
        int[] numbers = toInts(values);
        return sum(numbers);
    }

    private static int[] toInts(String[] values) { // Int로 변환만하는 함수
        int[] numbers = new Int[values.length];
        for (int i = 0; i < values.length; i++) {
            numbers[i] = Integer.parseInt(values[i]);
        }
        return numbers;
    }

    private static int sum(int[] numbers) {  // 더하기만 하는 함수
        int result = 0;
        for (int number: numbers) {
            result += number;
        }
        return result;
    }
}
```

이 코드 보자마자 든 생각이 포문 한번 더도는데 이게 정말 맞아? 생각하고 있었는데 바로 강사분께서 서비스에서 for문 돌리는 데이터는 대부분 크지 않기 때문에 성능에 아무 영향이 없다고 한다.

메서드가 단 하나의 일만하므로 재활용하는데에 제약이 있다. 재활용을 위해 반드시 하나만 해야한다. 테스트를 위해서는 더더욱 하나의 일만해야한다.

# 로컬 변수가 정말 필요한가?

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        if (text == null || text.isEmpty()) {
            result = 0;
        }
        String[] values = text.split(",|:");
        int[] numbers = toInts(values);
        return sum(numbers);
    }
}
```

values, numbers 로컬 변수가 필요없어보인다.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        if (text == null || text.isEmpty()) {
            result = 0;
        }

        return sum(toInts(text.split(",|:");););  // 이런식으로 줄일 수 있다.
    }
}
```

불필요한 로컬변수는 지워준다.

# compose method pattern

메서드(함수)의 의도가 잘 드러나도록 동등한 수준의 작업을 하는 여러 단계로 나눈다.

:추상화 레벨을 같게 한다.

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        if (isBlank(text)) { // 1단계 추상화
            result = 0;
        }

        return sum(toInts(split(text));  // 1단계 추상화
    }

    private static boolean isBlank(String text) {
        return text == null || text.isEmpty();
    }

    private static String[] split(String text) {
        return text.split(",|:");
    }

    private static int[] toInts(String[] values) {
        [...]
    }

    private static int sum(int[] numbers) {
        [...]
    }
}
```

첫 코드와 리팩토링 결과 코드를 비교해보자

## 리팩토링 전

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        int result = 0;
        if (text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            for (String value : values) {
                result += Integer.parseInt(value);
            }
        }
        return result;
    }
}
```

## 결과 코드

```java
public class StringCalculator {
    public static int splitAndSum(String text) {
        if (isBlank(text)) { // 1단계 추상화
            result = 0;
        }

        return sum(toInts(split(text));  // 1단계 추상화
    }

    private static boolean isBlank(String text) {
        return text == null || text.isEmpty();
    }

    private static String[] split(String text) {
        return text.split(",|:");
    }

    private static int[] toInts(String[] values) {
        [...]
    }

    private static int sum(int[] numbers) {
        [...]
    }
}
```

사람마다 다르겠지만 다른 사람이 이 두 코드를 받게 된다면 리팩토링 후의 코드가 더 보기 쉽다고 본다.
