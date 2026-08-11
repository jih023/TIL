## 추가 공부 
### 1. 책의 코드는 의사코드다
 
책에는 `List` 인터페이스의 `sort` 디폴트 메서드가 이렇게 소개된다.
 
```java
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
}
```
 
하지만 실제 JDK의 `Collections.sort`는 다음과 같다.
 
```java
public static <T> void sort(List<T> list, Comparator<? super T> c) {
    list.sort(c);
}
```
 
* `Collections.sort` → `list.sort` 를 호출한다.
* 만약 `List.sort`의 디폴트 구현이 다시 `Collections.sort`를 부른다면 **무한 재귀 → `StackOverflowError`** 가 발생한다.
* 즉 책의 코드는 "인터페이스에 메서드 본문이 들어간다"는 개념만 보여주기 위한 의사코드다.
자바 8 이전에는 반대 방향이었다. 정렬 로직이 `Collections.sort`에 있었고, 자바 8에서 `List.sort`가 생기면서 **정렬의 주인이 `Collections` → `List`로 옮겨가고, `Collections.sort`는 하위 호환용 얇은 위임 메서드로 남았다.**
 
### 2. 진짜 디폴트 구현
 
```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();          // 1. 배열로 복사
    Arrays.sort(a, (Comparator) c);       // 2. 배열을 정렬
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {                  // 3. 리스트에 되돌려 씀
        i.next();
        i.set((E) e);
    }
}
```
 
#### 왜 배열로 복사해서 정렬하는가?
 
* **어떤 `List` 구현체가 올지 알 수 없기 때문이다.**
* `LinkedList`처럼 랜덤 접근이 `O(n)`인 자료구조에 인덱스 기반 정렬(`get(i)` / `set(i, e)`)을 돌리면 성능이 최악이 된다.
* `ListIterator`를 이용한 순차 접근만으로 처리되는 형태를 택하면, 어떤 구현체가 와도 최소한의 성능이 보장된다.
* 대가는 **배열 복사 1회(추가 메모리 O(n))** 다.
#### 정렬 알고리즘
 
| 대상 | 알고리즘 | 안정 정렬 여부 |
| --- | --- | --- |
| `Arrays.sort(Object[])` | TimSort (병합 정렬 + 삽입 정렬) | O (안정) |
| `Arrays.sort(int[])` 등 기본형 | 듀얼 피벗 퀵소트 | X (불안정) |
 
* 객체 배열에 TimSort를 쓰기 때문에 **`List.sort`는 안정 정렬을 보장한다.**
* 기본형 배열은 값만 같으면 구분할 수 없으므로 안정성이 의미가 없고, 대신 메모리를 덜 쓰는 퀵소트를 쓴다.
### 3. ArrayList는 이 디폴트 메서드를 오버라이드한다
 
```java
@Override
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}
```
 
* 내부 배열 `elementData`를 **직접** 정렬한다. → `toArray()` 복사와 되쓰기 과정이 통째로 사라진다.
* `modCount` 검사: 정렬 중에 `Comparator`가 리스트를 수정하면 `ConcurrentModificationException`을 던진다.
* 반면 **`LinkedList`는 `sort`를 오버라이드하지 않는다.** 디폴트 구현(배열 복사 방식)이 이미 연결 리스트에 최적이기 때문이다.
### 4. 여기서 배울 디폴트 메서드의 설계 의도
 
디폴트 메서드는 단순히 "하위 호환을 위한 땜빵"이 아니다.
 
1. **안전한 기본값 제공** — 모든 구현체가 최소한 동작하도록 보장하는 범용 구현을 인터페이스에 둔다.
2. **최적화는 구현체의 몫** — 더 잘할 수 있는 구현체(`ArrayList`)는 오버라이드해서 성능을 낸다.
3. **하지 않아도 됨** — 디폴트로 충분한 구현체(`LinkedList`)는 그냥 상속받는다.
> 인터페이스가 "무엇을(what)"뿐 아니라 "합리적인 기본 동작(reasonable default how)"까지 제공하고,
> 구현체는 필요할 때만 "더 나은 how"로 덮어쓴다.
 
### 5. 직접 확인해보기
 
* IDE에서 `List.java`를 열어 `sort`를 찾는다. (IntelliJ: `Ctrl/Cmd + N` → `List`)
* `ArrayList.java`에서 같은 이름의 메서드를 찾아 나란히 비교한다.
* `LinkedList.java`에는 `sort`가 없다는 것을 확인한다.
```java
// 안정 정렬 확인 실험
record P(String name, int age) {}
 
List<P> list = new ArrayList<>(List.of(
    new P("a", 30), new P("b", 20), new P("c", 30), new P("d", 20)
));
list.sort(Comparator.comparingInt(P::age));
// 결과: b, d, a, c  → 같은 age 안에서 원래 순서(b→d, a→c)가 유지된다
```