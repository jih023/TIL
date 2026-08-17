# Chapter 2. 동작 파라미터화 코드 전달하기

## 2.0 동작 파라미터화 간보기
* 아직 어떻게 실행할 것인지 결정하지 않은 코드 블록
* 자주 바뀌는 요구사항에 효과적으로 대응할 수 있음

## 2.1 변화하는 요구사항 대응하기
1. 녹색 사과 필터링
   ```java
   // filterApplesByColor(List<Apple> inventory) 함수 내부
   if (GREEN.equals(apple.getColor())) {
       result.add(apple);
   }
   ```
   * 근데 만약 녹색 사과 말고 빨간 사과도 필터링하고 싶다면?
   * 메서드를 복제해서 빨간 사과 전용 메서드를 만든다
     * ⇒ 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.
2. 색을 파라미터화
   ```java
   // filterApplesByColor(List<Apple> inventory, Color color) 함수 내부
   if (apple.getColor().equals(color)) {
       result.add(apple);
   }
   ```
   * 농부는 아래처럼 사용할 수 있다.
     * `filterApplesByColor(inventory, GREEN);`
     * `filterApplesByColor(inventory, RED);`
   * 만약 농부가 색 이외에 무게로 구분하고 싶다면?
     * 다양한 무게에 대응할 수 있도록 무게 정보 파라미터를 추가해보자.
     ```java
     // filterApplesByWeight(List<Apple> inventory, int weight) 함수 내부
     if (apple.getWeight() > weight) {
         result.add(apple);
     }
     ```
     * 무게 필터링을 하는 부분이 색 필터링 코드랑 중복됨
       * ⇒ 소프트웨어 공학의 DRY(don't repeat yourself, 같은 것을 반복하지 말 것) 원칙을 어김
3. 가능한 모든 속성으로 필터링
   * 이건 너무….. 무식한 방법 같아서 눈으로만 읽음
   * 일단 함수 파라미터가 `(List<Apple> inventory, Color color, int weight, boolean flag)`
   * 필터링 요구사항이 바뀌었을 때 유연하게 대응할 수 없음

## 2.2 동작 파라미터화
* 선택 조건을 결정하는 인터페이스 정의 (참 / 거짓을 반환하는 함수를 프레디케이트라고 함)
  ```java
  public interface ApplePredicate {
      boolean test (Apple apple);
  }
  ```
* 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate 정의 가능
  * 무거운 사과 선택
    ```java
    public class AppleHeavyWeightPredicate implements ApplePredicate {
        public boolean test(Apple apple) {
            return apple.getWeight() > 150;
        }
    }
    ```
  * 녹색 사과 선택
    ```java
    public class AppleGreenColorPredicate implements ApplePredicate {
        public boolean test(Apple apple) {
            return GREEN.equals(apple.getColor());
        }
    }
    ```
* 이를 전략 디자인 패턴이라고 한다.
  * 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음, 런타임에 알고리즘을 선택하는 기법이다.
  * 위 코드에서는 ApplePredicate가 알고리즘 패밀리, AppleHeavyWeightPredicate와 AppleGreenColorPredicate가 전략이다.
  * ApplePredicate는 filterApples에서 ApplePredicate 객체를 받아 애플의 조건을 검사하도록 메서드를 고친다. 이걸 동작 파라미터화라고 한다.
  * 즉, 메서드가 다양한 동작(전략)을 받아서 내부적으로 다양한 동작을 수행할 수 있다.

1. 추상적 조건으로 필터링
   * ApplePredicate를 이용해서 필터 메서드를 수정한다.
     ```java
     // filterApples(List<Apple> inventory, ApplePredicate p) 함수 내부
     for(Apple apple: inventory) {
         if(p.test(apple)) { // 프레디케이트 객체로 사과 검사 조건을 캡슐화
             result.add(apple);
         }
     }
     ```
   * 농부가 만약 150그램이 넘는 빨간 사과를 검색해달라고 부탁하면 ApplePredicate를 적절하게 구현하는 클래스만 만들면 된다.
     ```java
     public class AppleRedAndHeavyPredicate implements ApplePredicate {
         public boolean test(Apple apple) {
             return RED.equals(apple.getColor()) && apple.getWeight() > 150;
         }
     }

     List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
     ```

### 퀴즈 2.1 유연한 prettyPrintApple 메서드 구현하기
* prettyPrintApple 메서드는 각각의 사과 무게를 출력하도록 지시할 수 있고, 각각의 사과가 무거운지, 가벼운지 출력하도록 지시할 수 있다. 아래는 예시 코드이다.
  ```java
  public static void prettyPrintApple(List<Apple> inventory, ???) {
      for(Apple apple: inventory) {
          String output = ???.???(apple);
          System.out.println(output);
      }
  }
  ```
* 내가 작성한 코드
  ```java
  public interface AppleFormatter {
      String accept(Apple apple);
  }

  public class AppleWeightFormatter implements AppleFormatter {
      public String accept(Apple apple) {
          return "this apple is " + apple.getWeight() + "g";
      }
  }

  public class AppleCommentFormatter implements AppleFormatter {
      public String accept(Apple apple) {
          if (apple.getWeight() > 150) {
              return "this apple is heavy";
          }
          else {
              return "this apple is light";
          }
      }
  }

  public static void prettyPrintApple(List<Apple> inventory, AppleFormatter formatter) {
      for(Apple apple: inventory) {
          String output = formatter.accept(apple);
          System.out.println(output);
      }
  }
  ```

## 2.3 복잡한 과정 간소화
* 현재 filterApples 메서드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의하고 인스턴스화해야 한다. 이를 개선하려고 한다.
* 자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 익명 클래스라는 기법을 제공한다. 다만 익명 클래스가 모든 것을 해결하는 것은 아니다.

1. 익명 클래스 사용
   * 익명 클래스를 이용해서 ApplePredicate를 구현하는 객체를 만드는 방법으로 필터링
     ```java
     List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
         public boolean test(Apple apple) {
             return RED.equals(apple.getColor());
         }
     });
     ```
   * 익명 클래스는 여전히 많은 공간을 차지한다는 단점이 있다. 그리고 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다.
2. 람다 표현식 사용
   ```java
   List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
   ```
   * 자바8의 람다를 사용하였더니 이전 코드보다 훨씬 간결해졌다!
3. 리스트 형식으로 추상화
   ```java
   public interface Predicate<T> {
       boolean test(T t);
   }

   public static <T> List<T> filter(List<T> list, Predicate<T> p) {
       List<T> result = new ArrayList<>();
       for(T e: list) {
           if(p.test(e)) {
               result.add(e);
           }
       }
       return result;
   }
   ```
   * 이렇게 되면 사과 외에도 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드 사용이 가능하다.
   ```java
   List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
   List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
   ```
   * 람다 표현식도 똑같이 사용 가능하다.

## 2.4 실전 예제

> **📌 2.3 다음에 왜 갑자기 2.4가 나오는가?**
>
> 2.4는 새로운 개념을 배우는 절이 아니라, 2.2~2.3에서 배운 걸 **자바 표준 API에서 이미 이렇게 쓰고 있다**고 확인시켜 주는 절이다.
>
> 2.1~2.3은 전부 사과 예제 하나로만 진행됐기 때문에 "이거 책이 지어낸 예제 아닌가?"라는 의심이 들 수 있다. 2.4는 그 의심에 답한다. 특히 2.3의 마지막에서 직접 만든 `Predicate<T>`는 사실 자바가 이미 갖고 있는 `java.util.function.Predicate`와 같다. 직접 만들어보게 한 다음 "사실 이미 있어"라고 알려주는 흐름의 연장선이다.
>
> 네 예제는 전부 **같은 틀**이다. → 메서드가 딱 하나인 인터페이스를 정의하고, 그 구현 객체를 메서드에 넘기고, 익명 클래스로 쓰다가, 람다로 줄인다.
>
> | 인터페이스 | 메서드 | 전달하는 "동작" |
> |---|---|---|
> | `ApplePredicate` (직접 만듦) | `test(Apple) → boolean` | 어떤 사과를 고를지 |
> | `Comparator<T>` | `compare(T, T) → int` | 무엇을 기준으로 정렬할지 |
> | `Runnable` | `run() → void` | 스레드에서 실행할 코드 |
> | `Callable<V>` | `call() → V` | 실행하고 결과까지 돌려줄 작업 |
> | `EventHandler<T>` | `handle(Event) → void` | 버튼이 눌렸을 때 할 일 |
>
> 첫 줄만 내가 만든 것이고 나머지는 자바가 제공하는 것인데 생김새가 같다. 정렬, 스레드, 비동기 작업, GUI 이벤트처럼 서로 상관없어 보이는 영역들이 알고 보면 전부 동작 파라미터화 패턴 위에 서 있다는 것이 2.4가 하려는 말의 전부다.
>
> 각 예제가 익명 클래스 버전과 람다 버전을 나란히 보여주는 것도 의도적이다. 람다를 계속 눈에 익혀서 3장(람다 표현식)으로 넘어가는 다리를 놓는 것이다.
>
> 그러므로 각 예제를 깊이 파기보다 "아, 이것도 같은 패턴이구나" 정도로 넘어가도 된다. `ExecutorService`나 자바FX는 여기서 처음 등장하는 데다 이 책의 주제도 아니다.

* Comparator로 정렬하기
  * `java.util.Comparator` 객체를 이용해서 sort 동작 파라미터화하기
    ```java
    public interface Comparator<T> {
        int compare(T o1, T o2);
    }
    ```
  * 익명 클래스를 이용해서 무게가 적은 순서로 목록에서 사과 정렬하기
    ```java
    inventory.sort(new Comparator<Apple>() {
        public int compare(Apple a1, Apple a2) {
            return a1.getWeight().compareTo(a2.getWeight());
        }
    });
    ```
  * 농부의 요구사항이 바뀌면 새로운 요구사항에 맞는 Comparator를 만들어 sort 메서드에 전달할 수 있다.
    ```java
    inventory.sort(
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
    ```
* Runnable로 코드 블록 실행하기
  * 자바 스레드를 이용하면 병렬로 코드 블록을 실행할 수 있는데, 여러 스레드가 각자 다른 코드를 실행하고, 나중에 실행할 수 있는 코드를 구현할 방법이 필요하다.
  * 자바 8까지는 Thread 생성자에 객체만을 전달할 수 있었으므로 void run 메소드를 포함하는 익명 클래스가 Runnable 인터페이스를 구현하도록 하는 것이 일반적인 방법이었다.
  * Runnable 인터페이스를 이용해 다양한 동작을 스레드로 실행할 수 있다.
    ```java
    public interface Runnable {
        void run();
    }

    Thread t = new Thread(new Runnable() {
        public void run() {
            System.out.println("Hello world");
        }
    });
    ```
  * 람다 표현식을 사용한 스레드 코드
    ```java
    Thread t = new Thread(() -> System.out.println("Hello world"));
    ```
* Callable을 결과로 반환하기
  * Runnable과 Callable의 차이는 딱 하나, **결과를 돌려주느냐 마느냐**다.
    * `Runnable.run()`은 반환 타입이 `void`다. "이 코드 좀 실행해줘"라고 던지고 끝이라 결과를 받을 방법이 없다.
    * `Callable<V>.call()`은 `V`를 반환한다. "이 작업 하고 결과 좀 줘"인 셈이다.
  * ExecutorService 추상화 개념
    * **스레드 풀 관리자**다. `new Thread()`를 직접 만들면 작업할 때마다 스레드를 생성하고 버리게 되는데 이게 비싸다.
    * ExecutorService는 스레드 몇 개를 미리 만들어두고 재사용하면서, 우리는 스레드가 아니라 **작업(태스크)만 던져주면** 알아서 배분해준다.
    * `Executors.newCachedThreadPool()`이 그 풀을 만드는 코드다.
  * `Future<V>`는 **"지금은 비어 있지만 나중에 결과가 들어올 상자"**다.
    * 다른 스레드에서 돌아가니까 결과가 즉시 나오지 않기 때문에, `submit()`은 값 대신 Future를 돌려준다.
    * 나중에 `threadName.get()`을 호출하면 그때 실제 값을 꺼낼 수 있고, 아직 안 끝났으면 끝날 때까지 기다린다.
  * Callable 인터페이스를 이용해 결과를 반환하는 태스크 만들기
    ```java
    public interface Callable<V> {
        V call();
    }

    ExecutorService executorService = Executors.newCachedThreadPool();
    Future<String> threadName = executorService.submit(new Callable<String>() {
        @Override
        public String call() throws Exception {
            return Thread.currentThread().getName();
        }
    });
    ```
  * 람다 표현식을 사용한 코드
    ```java
    Future<String> threadName = executorService.submit(
        () -> Thread.currentThread().getName());
    ```
* GUI 이벤트 처리하기
  * GUI 프로그래밍
    * 마우스 클릭이나 문자열 위로 이동하는 등의 이벤트에 대응하는 동작을 수행하는 식으로 동작한다.
  * GUI 프로그래밍에서도 변화에 대응할 수 있는 유연한 코드가 필요하다.
  * 동작 파라미터화가 왜 필요한지 가장 직관적으로 드러나는 사례다.
    * 버튼을 만드는 시점에는 그 버튼이 눌렸을 때 뭘 해야 하는지 버튼 자신은 알 수 없다. 어떤 버튼은 저장하고 어떤 버튼은 삭제할 것이다.
    * 그래서 "눌렸을 때 할 일"을 나중에 setOnAction으로 밖에서 주입해준다. 2.0에서 말한 "아직 어떻게 실행할 것인지 결정하지 않은 코드 블록"이 바로 이것이다.
  * 자바FX에서는 setOnAction 메서드에 EventHandler를 전달함으로써 이벤트에 어떻게 반응할지 설정할 수 있다.
    ```java
    Button button = new Button("Send");
    button.setOnAction(new EventHandler<ActionEvent>() {
        public void handle(ActionEvent event) {
            label.setText("Sent!!");
        }
    });
    ```
  * 람다 표현식을 사용한 코드
    ```java
    button.setOnAction((ActionEvent event) -> label.setText("Sent!!"));
    ```

## 2.5 마치며
* 동작 파라미터화 정리
  * 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
  * 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다.
  * 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다. 자바 8에서는 인터페이스를 상속받아 여러 클래스를 구현해야 하는 수고를 없앨 수 있는 방법을 제공한다.
  * 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화 할 수 있다.
