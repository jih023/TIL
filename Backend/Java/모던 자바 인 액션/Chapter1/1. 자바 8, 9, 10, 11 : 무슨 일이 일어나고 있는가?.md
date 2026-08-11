# Chapter 1. 자바 8, 9, 10, 11 : 무슨 일이 일어나고 있는가?

## 1. 자바의 병렬 처리 진화 과정
자바는 병렬 실행 환경을 쉽게 관리하고 에러가 덜 발생하는 방향으로 진화하려 노력해왔다.

* 자바 1.0 : 스레드, 락, 메모리 모델 지원
* 자바 5 : 스레드 풀, 병렬 실행 컬렉션
* 자바 7 : 포크/조인 프레임워크
* 자바 8 : 병렬 실행을 새롭고 단순한 방식으로 접근할 수 있는 방법을 제공
* 자바 9 : 리액티브 프로그래밍 기법(RxJava)

자바 8의 요구사항
1. 간결한 코드
2. 멀티코어 프로세서의 쉬운 활용

자바 8에서 제공하는 기능
1. 스트림 API
   * 데이터베이스 질의 언어처럼 고수준 언어로 원하는 동작을 표현하면, 구현(스트림)에서 최적의 저수준 실행 방법을 선택하는 방식으로 동작한다.
   * 스트림을 이용하면 에러를 자주 일으키며 멀티코어 CPU를 이용하는 것보다 비용이 훨씬 비싼 키워드 `synchronized`를 사용하지 않아도 된다.
2. 메서드에 코드를 전달하는 기법
3. 인터페이스의 디폴트 메서드

## 2. 왜 아직도 자바는 변화하는가?
자바가 대중적인 프로그래밍 언어로 성장한 배경에는 1990년대에 각광받은 객체지향이 있다.
* 캡슐화 덕분에 C에 비해 소프트웨어 엔지니어링적인 문제가 훨씬 적다.
* 객체지향의 정신적인 모델 덕분에 윈도우 95 및 그 이후의 WIMP 프로그래밍 모델에 쉽게 대응할 수 있었다.

스트림 처리
* 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이다.
* 자바 8에서 `java.util.stream` 패키지에 스트림 API가 추가되었다. → `Stream<T>`

동작 파라미터화(behavior parameterization)로 메서드에 코드 전달하기
* 메서드를 다른 메서드의 인수로 넘겨주는 기능이다.

병렬성과 공유 가변 데이터
* 스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 안전하게 실행돼야 한다. → 공유된 가변 데이터에 접근하지 않아야 한다.
* 이러한 함수를 순수 함수, 부작용 없는 함수, 상태 없는 함수라고 부른다.

자바 8의 가장 큰 변화
* 기존 값을 변화시키는 데 집중했던 고전적인 객체지향에서 벗어나, 함수형 프로그래밍으로 다가섰다.
* 함수형 프로그래밍에서는 우리가 하려는 작업이 최우선시되며, 그 작업을 어떻게 수행하는지는 별개의 문제로 취급한다.
* 자바 8에서 함수형 프로그래밍을 도입함으로써 두 가지 프로그래밍 패러다임(함수형, 객체지향)의 장점을 모두 활용할 수 있게 되었다.

## 3. 자바 함수
프로그래밍 언어에서 함수라는 용어는 메서드(정적 메서드)와 같은 의미로 사용된다.
자바의 함수는 이에 더해 수학적인 함수처럼 사용되며 부작용을 일으키지 않는 함수를 의미한다.

함수가 필요한 이유
* 메서드와 클래스는 그 자체로 값이 될 수 없기 때문이다.

### 3.1 메서드 참조
디렉터리에서 숨겨진 파일을 필터링한다고 가정하고, `File` 클래스의 `isHidden` 메서드를 사용해보자.

```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
    public boolean accept(File file) {
        return file.isHidden();
    }
});
```

* 3행의 코드지만 각 행이 무슨 작업을 하는지 투명하지 않다.
* `File` 클래스에 이미 `isHidden`이라는 메서드가 있는데 왜 굳이 `FileFilter`로 감싸고 `FileFilter`를 인스턴스화할까? → 자바 8 전까지는 방법이 없었기 때문이다.

```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

* 자바 8의 메서드 참조 `::`(이 메서드를 값으로 사용하라는 의미)를 이용해서 `listFiles`에 직접 전달할 수 있다.

### 3.2 람다 : 익명 함수
* 함수도 값으로 취급할 수 있다.
  * `(int x) -> x + 1` : x라는 인수로 호출하면 x + 1을 반환한다.
* 직접 메서드를 정의할 수도 있지만, 이용할 수 있는 편리한 클래스나 메서드가 없을 때 새로운 람다 문법을 이용한다.

### 3.3 코드 넘겨주기
```java
public static boolean isGreenApple(Apple apple) {
    return GREEN.equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple) {
    return apple.getWeight() > 150;
}

public interface Predicate<T> {
    boolean test(T t);
}

static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}

// 다음처럼 메서드 호출 가능
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyApple);
```

Predicate란?
* 앞선 예제에서 `filterApples`는 `Predicate<Apple>`을 파라미터로 받는다.
* 수학에서는 인수로 값을 받아 true나 false를 반환하는 함수를 프레디케이트라고 한다.
* `Function<Apple, Boolean>`처럼 구현할 수도 있지만, `Predicate<Apple>`을 사용하는 것이 더 표준적인 방식이다.
  * 또한 boolean을 Boolean으로 변환하는 과정이 없으므로 더 효율적이다.

### 3.4 메서드 전달에서 람다로
* `isGreenApple`, `isHeavyApple`처럼 한두 번 사용할 메서드를 매번 정의하는 것은 매우 귀찮다.
* 람다를 통해 구현할 수 있다.

```java
filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()));
filterApples(inventory, (Apple a) -> a.getWeight() > 150);
filterApples(inventory, (Apple a) -> a.getWeight() < 80 || RED.equals(a.getColor()));
```

## 4. 스트림
예를 들어 리스트에서 고가의 트랜잭션만 필터링한 다음 통화로 결과를 그룹화해야 하는 상황을 생각해보자.

```java
Map<Currency, List<Transaction>> transactionsByCurrencies =
    transactions.stream()
        .filter((Transaction t) -> t.getPrice() > 1000)  // 고가의 트랜잭션 필터링
        .collect(groupingBy(Transaction::getCurrency));  // 통화로 그룹화
```

* 컬렉션에서는 반복 과정을 직접 처리해야 했다. (for-each 루프를 이용해 각 요소를 반복하면서 작업을 수행)
  * 이러한 방식의 반복을 외부 반복이라고 한다.
* 스트림 API에서는 라이브러리 내부에서 모든 데이터가 처리된다.
  * 이러한 방식의 반복을 내부 반복이라고 한다.

멀티스레딩
* 포킹 단계 : 두 CPU를 가진 환경에서 리스트를 필터링할 때 한 CPU는 리스트의 앞부분을, 다른 CPU는 리스트의 뒷부분을 처리하도록 요청할 수 있다.
* 스트림과 람다 표현식을 이용하면 병렬성을 얻을 수 있다.
  * 순차 처리 방식 : `stream()`
  * 병렬 처리 방식 : `parallelStream()`

## 5. 디폴트 메서드와 자바 모듈
디폴트 메서드
* 구현 클래스에서 구현하지 않아도 되는 메서드를 인터페이스에 추가할 수 있는 기능을 제공한다. 메서드 본문은 클래스 구현이 아니라 인터페이스의 일부로 포함된다. → 디폴트 메서드라고 부르는 이유
* 자바 8에서는 `List`에 직접 `sort` 메서드를 호출할 수 있는데, `List` 인터페이스에 디폴트 메서드 정의가 추가되었기 때문이다.

```java
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
}
```

## 문제 
-  스트림에 넘기는 람다가 지켜야 하는 조건은? 
1.  반드시 예외를 던지지 않아야 한다.
2.  공유된 가변 데이터에 접근하지 않아야 한다.
3. 반드시 한 줄로 작성해야 한다.
4.  반드시 boolean을 반환해야 한다. 

- 람다에서 지역 변수를 캡처할 때 그 변수가 사실상 final(effectively final)이어야 하는 이유는? 
1. 문법을 단순하게 만들기 위한 관례이다.
2. 지역 변수는 스택에 있어 람다가 다른 스레드에서 실행되면 이미 사라졌을 수 있기 때문이다.
3. final 변수가 메모리를 덜 쓰기 때문이다. 
4. 컴파일러가 final 변수만 인식할 수 있기 때문이다. 