# LSP (Liskov Substitution Principle)
- 서브 타입은 언제나 기반(부모) 타입으로 교체할 수 있어야 한다는 원칙

> ✅ **교체하다** : 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행이 보장되어야 한다는 의미

- 즉, 부모 클래스의 인스턴스를 사용하는 위치에 자식 클래스의 인스턴스를 대신 사용했을 때 코드가 원래 의도대로 작동해야 한다.
  - 이것을 부모 클래스와 자식 클래스 사이의 행위가 일관성이 있다고 말한다.


#### 자바의 컬렉션 프레임워크 예시
```java
import java.util.Collection;
import java.util.HashSet;
import java.util.LinkedList;

public class Main {
    public static void main(String[] args) {
        // Collection 인터페이스 타입으로 변수 선언
        Collection<Integer> data = new LinkedList<>();
        data.add(1);
        data.add(2);

        // 중간에 전혀 다른 자료형 클래스를 할당해도 호환됨
        data = new HashSet<>();

        // 인터페이스 구현 구조가 잘 잡혀있기 때문에 add 메소드 동작이 각기 자료형에 맞게 보장됨
        data.add(4);
        data.add(5);
        
        for (Integer value : data) {
            System.out.print(value + " "); // 4 5
        }
    }
}
```
- 인터페이스 Collection의 추상 메서드를 각기 하위 자료형 클래스에서 implements하여 인터페이스 구현 규약을 잘 지키도록 미리 잘 설계되어 있다.

<img src="https://velog.velcdn.com/images/dev-baik/post/72b26821-e130-4684-ac2b-d8c3a75bb2b5/image.png"/>

> **리스코프 치환 원칙(LSP)은 다형성을 지원하기 위한 원칙이다!**

# LSP 원칙 위반예제
- 리스코프 치환 원칙의 핵심 : 부모 클래스의 행동 규약을 자식 클래스가 위반하면 안된다.

> ✅ **행동 규약을 위반하다** : 자식 클래스가 오버라이딩을 할 때, 잘못되게 재정의하면 리스코프 치환 원칙을 위배할 수 있다는 의미이다.

#### 자식 클래스가 오버라이딩을 잘못하는 경우
1. 자식 클래스가 부모 클래스의 메서드 시그니처를 자기 멋대로 변경하는 경우
2. 자식 클래스가 부모 클래스의 의도와 다르게 메서드를 오버라이딩하는 경우

## 자식의 잘못된 메서드 오버로딩
```kotlin
open class Animal {
    open var speed = 100

    open fun go(distance: Int): Int {
        return speed * distance
    }
}

class Eagle : Animal() {
    // 오류 발생 : 부모 클래스의 행동 규약을 어김
    override fun go(distance: Int, flying: Boolean): String {
        return if (flying) {
            "$distance 만큼 날아서 갔습니다."
        } else {
            "$distance 만큼 걸어서 갔습니다."
        }
    }
}

fun main() {
    val eagle: Animal = Eagle()
    (eagle as Eagle).go(10, true)
}
```

## 부모의 의도와 다르게 메소드 오버라이딩
```kotlin
open class NaturalType(animal: Animal) {
    // 생성자로 동물 이름이 들어오면, 정규표현식으로 매칭된 동물 타입을 설정한다.
    val type = when (animal) {
        is Cat -> "포유류"
        else -> // ...
    }

    fun print(): String {
        return "이 동물의 종류는 $type 입니다."
    }
}

open class Animal {
    open fun getType(): NaturalType {
        return NaturalType(this)
    }
}

class Cat : Animal()

fun main() {
    val cat = Cat()
    val result = cat.getType().print()
    println(result) // 이 동물의 종류는 포유류 입니다.
}
```

#### 참고 : OCP 원칙을 준수하도록 변경
- **문제점** : 새로운 동물 타입을 추가하려면 NaturalType 클래스를 직접 수정해야 하므로 OCP 원칙을 위반한다.
```kotlin
open class NaturalType {
    open fun getType(): String {
        return //...
    }
}

class CatNaturalType : NaturalType() {
    override fun getType(): String {
        return "포유류"
    }
}

open class Animal {
    open fun getType(): NaturalType {
        return NaturalType()
    }
}

class Cat : Animal() {
    override fun getType(): NaturalType {
        return CatNaturalType()
    }
}

fun main() {
    val cat = Cat()
    val result = cat.getType().getType()
    println("이 동물의 종류는 $result 입니다.")
}
```

#### 다시 이어서
- **문제 상황** : 다른 개발자가 자식 클래스에 부모 메서드인 getType()의 반환값을 null로 오버라이딩 설정하여 메서드를 사용하지 못하게 설정하고, 대신 getName()이라는 메서드를 만들어 한번에 출력하도록 설정한다면 기존 코드는 예외가 발생하게 된다.
```kotlin
class Cat : Animal() {
    override fun getType(): NaturalType? {
        return null
    }

    fun getName(): String {
        return "이 동물의 종류는 포유류 입니다."
    }
}

fun main() {
    val cat = Cat()
    val result = cat.getType().print()
    println(result) // Error
}
```
- **HOW)** 사전에 약속한 기획대로 구현하고, 상속 시 부모에서 구현한 원칙을 따라야 한다.

## 잘못된 상속 관계 구성으로 인한 메서드 정의
- Animal 이라는 추상 클래스를 정의하고 동물은 낼 수 있기 때문에 추상메소드 speak()를 통하여 메서드 구현을 강제하도록 규칙을 지정하였다.
```kotlin
abstract class Animal {
    open fun speak() {}
}

class Cat : Animal() {
    override fun speak() {
        println("냐옹")
    }
}

class Dog : Animal() {
    override fun speak() {
        println("멍멍")
    }
}
```
- **문제 상황** : Fish 물고기 클래스에 Animal 추상 클래스를 상속하면, 물고기는 행할 수 없는 speak() 메서드를 구현해야 한다.
- **HOW)** Fish 클래스의 speak() 메서드는 동작을 하지 못하게 하고 예외를 던지도록 설정한다.
```kotlin
class Fish : Animal() {
    override fun speak() {
        try {
            throw Exception("물고기는 말할 수 없음")
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```
- **BUT)** 다른 개발자와 협스업할 때 제대로된 [스펙 문서](https://brunch.co.kr/@hj-kang/2)를 전달받지 못한다면 잘 동작하던 코드가 갑자기 예외를 던질 수 있다.

> ✅ **스펙 문서** : 무엇을, 왜 만들어야 하는지에 대한 명확한 결정사항을 모아놓은 문서

```kotlin
val list = mutableListOf<Animal>()
list.add(Cat())
list.add(Dog())
list.add(Fish())

for (a in list) {
    a.speak()
}
```
- **결과)** LSP 원칙에 따르면 speak() 메서드를 실행하면 각 동물 타입에 맞게 울부짖는 결과를 내보내야 되는데, 갑자기 뜬금없이 예외를 던져버리니 개발자 간 상호 신뢰를 잃게 될 수 있다.

> 리스코프 치환 원칙(LSP)은 협업하는 개발자 사이의 신뢰를 위한 원칙이기도 하다.

- **HOW)** 인터페이스로 분리하는 작업을 통해 수정해야 한다.
```kotlin
abstract class Animal

interface Speakable {
    fun speak()
}

class Cat : Animal(), Speakable {
    override fun speak() {
        println("냐옹")
    }
}

class Dog : Animal(), Speakable {
    override fun speak() {
        println("멍멍")
    }
}

class Fish : Animal()
```

# LSP 원칙 적용 주의점

> **리스코프 치환 원칙** : 다형성의 특징을 이용하기 위해 상위 클래스 타입으로 객체를 선언하여 하위 클래스의 인스턴스를 받으면, 업캐스팅된 상태에서 부모의 메서드를 사용해도 동작이 의도대로만 흘러가도록 구성하면 되는 것이다.


- 그리고 LSP 원칙의 핵심은 상속(Inheritance)이다.
- 그런데 주의할 점은, 객체 지향 프로그래밍에서 상속은 기반 클래스와 서브 클래스 사이에 `is-a 관계`가 있을 경우로만 제한되어야 한다.


- 그 외의 경우에는 합성(composition)을 이용하도록 권고되어 있다.


- 따라서 다형성을 이용하고 싶다면 extends 대신 인터페이스로 implements 하여 인터페이스 타입으로 사용하기를 권장하고, 상위 클래스의 기능을 이용하거나 재사용을 하고 싶다면 상속 보단 합성으로 구성하기를 권장한다.

#### 참고한 사이트
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-LSP-%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84-%EC%B9%98%ED%99%98-%EC%9B%90%EC%B9%99