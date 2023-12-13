# OCP (Open Closed Principle)
- 기존의 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계가 되어야 한다는 원칙
- 확장에 대해서는 개방적(open)이고, 수정에 대해서는 폐쇄적(closed)이어야 한다.
- [다형성](https://velog.io/@dev-baik/%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D#%EB%8B%A4%ED%98%95%EC%84%B1)과 확장을 가능케 하는 객체지향의 장점을 극대화하는 설계 원칙

## 확장에 열려있다
- 모듈의 확장성을 보장하는 것을 의미한다.
- 새로운 변경 사항이 발생했을 때 유연하게 코드를 추가함으로써 애플리케이션의 기능을 큰 힘을 들이지 않고 확장할 수 있다.

## 변경에 닫혀있다
- 객체를 직접적으로 수정하는 건 제한해야 한다는 것을 의미한다.
- 새로운 변경 사항이 발생했을 때 객체를 직접적으로 수정해야 한다면 새로운 변경사항에 대해 유연하게 대응할 수 없는 애플리케이션이라고 말한다.

# OCP 원칙 위반예제
## 기존 코드
```kotlin
class Animal(val type: String)

// 동물 타입을 받아 각 동물에 맞춰 울음소리를 내게 하는 클래스 모듈
class HelloAnimal {
    fun hello(animal: Animal) {
        when (animal.type) {
            "Cat" -> println("냐옹")
            "Dog" -> println("멍멍")
            else -> println("알 수 없는 동물")
        }
    }
}

fun main() {
    val hello = HelloAnimal()

    val cat = Animal("Cat")
    val dog = Animal("Dog")

    hello.hello(cat) // 냐옹
    hello.hello(dog) // 멍멍
}

```

## 요구사항 업데이트
```kotlin
fun main() {
    val hello = HelloAnimal()

    val cat = Animal("Cat")
    val dog = Animal("Dog")
    val sheep = Animal("Sheep")
    val lion = Animal("Lion")

    hello.hello(cat) // 냐옹
    hello.hello(dog) // 멍멍
    hello.hello(sheep)
    hello.hello(lion)
}
```
```kotlin
class HelloAnimal {
    // 기능을 확장하기 위해서는 클래스 내부 구성을 일일히 수정해야 하는 번거로움이 생긴다.
    fun hello(animal: Animal) {
        when (animal.type) {
            "Cat" -> println("냐옹")
            "Dog" -> println("멍멍")
            "Sheep" -> println("메에에")
            "Lion" -> println("어흥")
            // ...
            else -> println("알 수 없는 동물")
        }
    }
}
```
- **문제점** : 고양이와 개 외에 양이나 사자를 추가하면 각 객체의 필드 변수에 맞게 when문을 분기하여 구성해줘야 한다.
- **HOW)** 추상화 클래스를 구성하고 이를 상속하여 확장시키는 관계로 형성하기

# OCP 원칙 준수예제
#### 규칙
1. 변경(확장)될 것과 변하지 않은 것을 엄격히 구분한다.
2. 두 모듈이 만나는 지점에 추상화(추상클래스 or 인터페이스)를 정의한다.
3. 구현체에 의존하기보다 정의한 추상화에 의존하도록 코드를 작성한다.

```kotlin
// 추상클래스를 상속만 하면 메소드 강제 구현 규칙으로 규격화만 하면 확장에 제한 없다 (opened)
abstract class Animal {
    abstract fun speak()
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

class Sheep : Animal() {
    override fun speak() {
        println("매에에")
    }
}

class Lion : Animal() {
    override fun speak() {
        println("어흥")
    }
}

// 기능 확장으로 인한 클래스가 추가되어도, 더이상 수정할 필요가 없어진다 (closed)
class HelloAnimal {
    fun hello(animal: Animal) {
        animal.speak()
    }
}

fun main() {
    val hello = HelloAnimal()

    val cat: Animal = Cat()
    val dog: Animal = Dog()

    val sheep: Animal = Sheep()
    val lion: Animal = Lion()

    hello.hello(cat) // 냐옹
    hello.hello(dog) // 멍멍
    hello.hello(sheep) // 매에에
    hello.hello(lion) // 어흥
}
```

# OCP 원칙 적용 주의점
- 확장에는 열려있고 변경에는 닫히게 하기 위해서는 추상화를 잘 설계할 필요성이 있는데, 추상화(추상 클래스 or 인터페이스)를 정의할 때 여러 경우의 수에 대한 고려와 예측이 필요하다.

> 추상화란 다른 모든 종류의 객체로부터 식별될 수 있는 객체의 본질적인 특징이다. (feat. Grady Booch)

- 즉, 추상 메서드 설계에서 적당한 추상화 레벨을 선택함으로써, 어떠한 행위에 대한 본질적인 정의를 서브 클래스에 전파함으로써 관계를 성립되게 하는 것이다.


- 만일 이러한 추상화에 따른 상속 구조를 처음부터 이상하게 구성하게 되면, LSP과 ISP 위반으로 이어지게 된다. 또한 OCP는 DIP의 설계 기반이 되기도 한다.

#### 참고한 사이트
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-OCP-%EA%B0%9C%EB%B0%A9-%ED%8F%90%EC%87%84-%EC%9B%90%EC%B9%99