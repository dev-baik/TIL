## 함수형 인터페이스 활용

> ✅ **코틀린 - 함수형 인터페이스** : 추상 메서드가 단 하나만 존재하는 인터페이스를 의미하며, SAM(Single Abstract Method)라고도 한다. 이렇게 정의된 인터페이스는 람다 표현식이나 메소드 참조를 통해 인스턴스가 가능하므로, 코드를 더욱 간결하게 유지할 수 있다.

1. 부모 클래스의 자원을 상속받아 재정의하여 사용할 자식 클래스가 한번만 사용되고 버려질 자료형이면, 굳이 상단에 클래스를 정의하기보다는, 지역 변수처럼 익명 클래스로 정의하고 스택이 끝나면 삭제되도록 하는 것이 유지보수면에서나 프로그램 메모리면에서나 이점을 얻을 수 있다.


2. 클래스 중에는 단 한번만 객체로 생성하고 더 이상 재사용되지 않는 클래스가 있다. 예를 들어 이벤트 처리, 스레드 객체는 보통 프로그램에서 단 한번만 사용된다. 만약 이러한 객체의 클래스를 클래스 파일로 보관한다면 유지보수 측면에서 좋지 않다. 따라서 이런 클래스는 익명 클래스로 만드는 것이 좋다.

> **Q1. 익명 클래스로 만드는 것이 왜 좋을까?** : 클래스의 선언과 객체의 생성을 동시에 할 수 있으며, 메서드 내에서 기존에 존재하는 클래스를 일회용으로 구성하여 필요한 메서드를 재정의하여 사용할 수 있다.

> ✅ **일회용** : 해당 클래스의 인스턴스가 한 번만 생성되고, 이후에는 재사용되지 않는다. 해당 객체가 더 이상 참조되지 않으면 가비지 컬렉터에 의해 처리된다.

```java
// OnClickListener 인터페이스는 onClick이라는 메서드만 선언된 인터페이스다.
public interface OnClickListener {
    void onClick(View v);
}

// Button 클래스는 setOnClickListener 메소드를 사용해 버튼의 리스너를 설정한다. 
// 이때 인자의 타입은 OnClickListener다.
public class Button {
    public void setOnClickListener(OnClickListener l) { ... }
}
```
- 자바 8 이전의 자바에서는 setOnClickListener 메소드에게 인자로 넘기기 위해 [무명 클래스의 인스턴스](https://velog.io/@dev-baik/inner-class#%EC%9D%B5%EB%AA%85-%ED%81%B4%EB%9E%98%EC%8A%A4)를 만들어야만 했다.
```java
button.setOnClickListener(new OnclickListener() {
    @Override
    public void onClick(View v) {
        ...
    }
})
```
- 코틀린에서는 무명 클래스 인스턴스 대신 람다를 넘길 수 있다.
```kotlin
button.setOnClickListener { view -> ... }
```

## 람다 vs 무명 객체
람다는 보통 무명 클래스로 컴파일되고, 외부 변수를 캡처하지 않는 경우(외부 스코프에 의존하지 않는 경우), 람다 인스턴스는 한 번만 생성되고 재사용할 수 있다. 하지만 무명 객체는 매번 새로운 인스턴스를 생성한다. 

하지만, 람다가 외부 변수를 캡처한다면 람다가 생성되는 시점마다 새로운 무명 객체가 생성된다. 이런 경우는 실행 시점에 무명 클래스의 생성에 부가 비용이 들어서 일반 함수를 사용한 구현보다 비효율적이다.

> ✅ **클로저(Closure)** : 람다식으로 표현된 내부 함수에서 외부 범위에 선언된 변수에 접근할 수 있는 구조로, 실행 시점에서 람다식의 모든 참조가 포함된 닫힌(closed) 객체를 람다 코드와 함께 저장한다.
> 
> ✅ **캡처(Capture)** : 클로저가 외부 스코프의 변수를 잡아내는 과정을 의미한다. 캡처된 변수는 람다가 활성화되는 동안 생존하며, 람다는 외부 스코프와 상호작용할 수 있다.

```java
// 함수형 인터페이스(Runnable)를 인자로 원하는 자바 메소드에 코틀린 람다를 전달할 수 있다.
void postponeComputation(int delay, Runnable computation);
```

- 컴파일러는 자동으로 무명 클래스와 인스턴스를 만들어준다. 이때 그 무명 클래스에 있는 유일한 추상 메소드를 구현할 때 람다 본문을 메소드 본문으로 사용한다.
```kotlin
postponeComputation(1000) { println(42) } 
```
````kotlin
postponeComputation(1000, object: Runnable {
    override fun run() {
        println(42)
    }
})
````
- 전역 변수로 컴파일되는 람다는 프로그램 안에 단 하나의 인스턴스만 존재한다.
```kotlin
val runnable = Runnable { println(42) }

fun handleComputation() {
    // 모든 handleComputation 호출에 같은 객체를 사용한다.
    postponeComputation(1000, runnable)
}
```
- 람다가 주변 영역의 변수를 포획한다면 매 호출마다 같은 인스턴스를 사용할 수 없다. 그런 경우 컴파일러는 매번 주변 영역의 변수를 포획한 새로운 인스턴스를 생성해준다.
```kotlin
fun handleComputation(id: String) {
    // handleComputation을 호출할 때마다 새로 Runnable 인스턴스를 만든다.
    postponeComputation(1000, runnable)
}
```
## 인라인 함수
호출 지점에 함수의 코드가 삽입되어 실행되는 함수로, 함수 호출의 오버헤드를 제거하고, 별도의 스택 프레임이 생성되지 않아 스택 메모리 할당과 제거의 오버헤드도 감소한다.

inline 키워드 어떤 함수에 붙이면 컴파일러는 그 함수를 호출하는 코드를 함수 본문에 해당하는 바이트코드로 변환해준다.

```kotlin
fun nonInlined(block: () -> Unit) {
    block()
}

fun doSomething() {
    nonInlined { println("do something") }
}
```
- 위 코드를 자바로 변경 : [람다와 invoke의 관계](https://velog.io/@dev-baik/invoke-%ED%95%A8%EC%88%98#%EB%9E%8C%EB%8B%A4%EC%99%80-invoke%EC%9D%98-%EA%B4%80%EA%B3%84)
```java
public void nonInlined(Function0 block) {
    block.invoke();
}

public void doSomething() {
    noInlined(System.out.println("do something");
}
```
- 자바로 컴파일된 코드
```java
public static final void doSomething() {
    nonInlined(new Function() {
        public final void invoke() {
            System.out.println("do something");
        }
    });
}
```
- **문제점** : nonInlined 함수의 파라미터로 새로운 객체를 생성하여 넘겨주어 doSomething 함수를 호출할 때마다 새로운 객체가 생성된다.  
- **해결책** : 인라인 함수를 사용하여 그 함수를 호출하는 코드를 함수 본문에 해당하는 바이트코드로 변환해준다.
  - 즉, 항상 새로운 객체를 생성하는 것이 아니라 해당 함수의 내용을 호출 지점에 넣는 방식으로 컴파일 코드를 작성하게 된다.

```kotlin
inline fun inlined(block: () -> Unit) {
    block()
}

fun doSomething() {
    inlined { println("do something") }
}
```
- 자바로 컴파일된 코드
```java
public static final void doSomething() {
    System.out.println("do something");
}
```
- inline 키워드를 통해 위와 같이 불필요한 객체를 생성하지 않고 내부에서 사용되는 함수가 호출하는 함수(doSomething) 내부에 삽입된다.
- **주의** : 람다를 파라미터로 받는 함수의 경우는 인라이닝을 하는게 훨씬 유리하지만, 코드 크기가 큰 함수의 경우는 모든 호출 지점에 바이트코드가 복사되기 때문에 오히려 더 성능이 악화될 수 있다. 

#### 참고한 사이트
- https://gunjoon.tistory.com/154
- https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9D%B5%EB%AA%85-%ED%81%B4%EB%9E%98%EC%8A%A4Anonymous-Class-%EC%82%AC%EC%9A%A9%EB%B2%95-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0
- https://velog.io/@changhee09/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%B8%EB%9D%BC%EC%9D%B8-%ED%95%A8%EC%88%98