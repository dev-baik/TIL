# SOLID 5원칙
객체지향 설계에서 지켜줘야 할 5개의 소프트웨어 개발 원칙

> - SRP (Single Responsibility Principle) : 단일 책임 원칙
> - OCP (Open Closed Principle) : 개방 폐쇄 원칙
> - LSP (Liskov Substitution Principle) : 리스코프 치환 원칙
> - ISP (Interface Segregation Principle) : 인터페이스 분리 원칙
> - DIP (Dependency Inversion Principle) : 의존 역전 원칙

**좋은 설계** : 시스템에 새로운 요구사항이나 변경사항이 있을 때, 영향을 받는 범위가 적은 구조를 말한다. 그래서 시스템에 예상하지 못한 변경사항이 발생하더라도, 유연하게 대처하고 이후에 확장성이 있는 시스템 구조를 만들 수 있다.

#### SOLID 객체 지향 원칙을 적용하는 이유 : 좋은 설계를 위해서
코드를 확장하고 유지 보수 관리하기가 더 쉬워지며, 불필요한 복잡성을 제거해 리팩토링에 소요되는 시간을 줄임으로써 프로젝트 개발의 생산성을 높일 수 있다.

## 단일 책임 원칙 - SRP
- **WHAT)** 프로그램의 유지보수 성을 높이기 위한 설계 기법으로, 클래스(객체)는 단 하나의 책임(기능)만 가져야 한다는 원칙
  - 즉, 하나의 클래스는 하나의 기능을 담당하여 하나의 책임을 수행하는 데 집중되도록 클래스를 따로따로 여러개 설계하라는 원칙


- **WHY)** 하나의 클래스에 기능(책임)이 여러개 있다면 기능 변경(수정)이 일어났을 때 수정해야할 코드가 많아진다.
  - 즉, SRP 원칙을 따름으로써 한 책임의 변경으로부터 다른 책임의 변경으로의 연쇄작용을 극복할 수 있게 된다.

> 이때 책임의 범위는 딱 정해져있는 것이 아니고, 어떤 프로그램을 개발하느냐에 따라 개발자마다 생각 기준이 달라질 수 있다. 따라서 단일 책임 원칙에 100% 해답은 없다.

## 개방 폐쇄 원칙 - OCP
- 확장에 열려있어야 하며, 수정에 닫혀있어야 한다.
- 기능 추가 요청이 오면 클래스 확장을 통해 손쉽게 구현하면서, 확장에 따른 클래스 수정은 최소화 하도록 프로그램을 작성해야 하는 설계 기법

> ✅ **확장에 열려있다** : 새로운 변경 사항이 발생했을 때 유연하게 코드를 추가함으로써 큰 힘을 들이지 않고 애플리케이션의 기능을 확장할 수 있음.
> 
> ✅ **변경에 닫혀있다** : 새로운 변경 사항이 발생했을 때 객체를 직접적으로 수정을 제한함.

- 추상화 사용을 통한 관계 구축을 권장한다. 즉, 다형성과 확장을 가능케 하는 객체지향의 장점을 극대화하는 기본적인 설계 원칙

- OCP 원칙을 지키지 않는다면 when 문에서 새로운 타입을 추가하거나 다운캐스팅이 필요한 상황이 발생한다.
```kotlin
sealed class Shape

class Circle(val radius: Double) : Shape()

class Rectangle(val width: Double, val height: Double) : Shape()

fun calculateArea(shape: Shape): Double {
    return when (shape) {
        is Circle -> calculateCircleArea(shape)
        is Rectangle -> calculateRectangleArea(shape)
        // 새로운 타입이 추가될 때마다 when 문을 수정해야 함
    }
}

fun calculateCircleArea(circle: Circle): Double {
    return Math.PI * circle.radius * circle.radius
}

fun calculateRectangleArea(rectangle: Rectangle): Double {
    return rectangle.width * rectangle.height
}

fun main() {
    val circle = Circle(5.0)
    val rectangle = Rectangle(3.0, 4.0)

    println("Circle Area: ${calculateArea(circle)}")
    println("Rectangle Area: ${calculateArea(rectangle)}")
}
```
- OCP 원칙을 지켰을 경우 : 새로운 기능을 추가할 때 기존 코드를 수정하지 않아야한다.
```kotlin
sealed class Shape

class Circle(val radius: Double) : Shape()

class Rectangle(val width: Double, val height: Double) : Shape()

interface AreaCalculator {
    fun calculateArea(shape: Shape): Double
}

class CircleAreaCalculator : AreaCalculator {
    override fun calculateArea(shape: Shape): Double {
        if (shape !is Circle) {
            throw IllegalArgumentException("Shape must be a Circle")
        }
        return Math.PI * shape.radius * shape.radius
    }
}

class RectangleAreaCalculator : AreaCalculator {
    override fun calculateArea(shape: Shape): Double {
        if (shape !is Rectangle) {
            throw IllegalArgumentException("Shape must be a Rectangle")
        }
        return shape.width * shape.height
    }
}

fun main() {
    val circle = Circle(5.0)
    val rectangle = Rectangle(3.0, 4.0)

    val circleAreaCalculator = CircleAreaCalculator()
    val rectangleAreaCalculator = RectangleAreaCalculator()

    println("Circle Area: ${circleAreaCalculator.calculateArea(circle)}")
    println("Rectangle Area: ${rectangleAreaCalculator.calculateArea(rectangle)}")
}
```

## 리스코프 치환 원칙 - LSP
- 서브 타입은 언제나 기반(부모) 타입으로 교체할 수 있어야 한다는 원칙
- 다형성의 특징을 이용하기 위해 상위 클래스 타입으로 객체를 선언하여 하위 클래스의 인스턴스를 받으면, 업캐스팅된 상태에서 부모의 메서드를 사용해도 동작이 의도대로 흘러가야 하는 것을 의미한다.

> 부모 메서드의 오버라이딩을 조심스럽게 따져가며 해야한다. 부모 클래스와 동일한 수준의 선행 조건을 기대하고 사용하는 프로그램 코드에서 예상치 못한 문제를 일으킬 수 있다.

## 인터페이스 분리 원칙 - ISP
- 인터페이스를 각각 사용에 맞게 끔 잘게 분리해야한다는 설계 원칙


- SRP 원칙이 클래스의 단일 책임을 강조한다면, ISP는 인터페이스의 단일 책임을 강조하는 것
- 즉, SRP 원칙의 목표는 클래스 분리를 통하여 이루어진다면, ISP 원칙은 인터페이스 분리를 통해 설계하는 원칙


- **목표** : 인터페이스를 사용하는 클라이언트를 기준으로 분리함으로써, 클라이언트의 목적과 용도에 적합한 인터페이스 만을 제공하는 것

> 한번 인터페이스를 분리하여 구성해놓고 나중에 무언가 수정사항이 생겨서 또 인터페이스를 분리하는 행위를 가지지 말아야 한다. (인터페이스는 한번 구성하였으면 웬만해선 변하면 안되는 정책 개념)

> 인터페이스는 제약 없이 자유롭게 다중 상속(구현)이 가능하기 때문에, 분리할 수 있으면 분리하여 각 클래스 용도에 맞게 구현하라는 설계 원칙

## 의존 역전 원칙 - DIP
- 어떤 클래스를 참조해서 사용해야하는 상황이 생긴다면, 그 클래스를 직접 참조하는 것이 아니라 그 대상의 상위 요소(추상 클래스 or 인터페이스)로 참조하라는 원칙
- 즉, 구현 클래스에 의존하지 말고, 인터페이스에 의존하라는 뜻
- 의존 관계를 맺을 때 변화하기 쉬운 것 또는 자주 변화하는 것보다는, 변화하기 어려운 것(거의 변화가 없는 것)에 의존하라는 것
- **지향점** : 각 클래스간의 결합도를 낮추는 것

#### 참고한 사이트
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%EC%84%A4%EA%B3%84%EC%9D%98-5%EA%B0%80%EC%A7%80-%EC%9B%90%EC%B9%99-SOLID
- https://velog.io/@haero_kim/SOLID-%EC%9B%90%EC%B9%99-%EC%96%B4%EB%A0%B5%EC%A7%80-%EC%95%8A%EB%8B%A4