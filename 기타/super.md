## super
- 자식 클래스는 부모 클래스를 상속받았기 때문에 자유롭게 부모의 모든 프로퍼티를 사용할 수 있다.
  - 하지만 자식 클래스가 부모 클래스의 프로퍼티와 동일한 이름을 갖고 있다면 그것을 부모로부터 구분해 낼 수 있어야 한다.
- super 키워드는 부모 클래스로부터 상속받은 필드나 메소드를 자식 클래스에서 참조하는 데 사용하는 참조 변수이다.
```java
class Parent {
    int a = 10;
}

class Child extends Parent {
    int a = 20;
    
    void display() {
        // a = 자식의 변수
        // 만약 자식에게 a라는 변수가 없었다면 부모의 a를 가리킴
        System.out.println(a); // 20
        System.out.println(this.a); // 20

        // 참조 변수 super를 통해 부모의 a에 접근 가능
        System.out.println(super.a); // 10
    }
}

public class Main {
    public static void main(String[] args) {
        Child ch = new Child();
        ch.display();
    }
}
```
- 다중 상속의 모호성 : 만약 Child 클래스가 두 개의 부모 클래스를 상속받는다면 어떤 멤버를 상속받아야 할지 알 수 없다.

### super()
- 자식 클래스가 인스턴스를 생성하면, 인스턴스 안에는 자식 클래스의 고유 멤버 뿐만 아니라 부모 클래스의 모든 멤버까지 포함되어 있다.
  - 하지만 생성자는 상속되지 않는다. 따라서 super() 메소드를 통해 부모 클래스의 생성자를 호출해야한다.
- 컴파일러는 컴파일 시 클래스에 생성자가 하나도 정의되지 않았다면 자동으로 기본 생성자를 추가해준다.
  - 하지만 부모 클래스에 기본 생성자가 아닌 매개변수를 가지는 생성자가 있다면(부모 클래스에서 생성자가 오버로딩되면) 자동으로 추가되지 않는다.
```java
class Parent {
    int a;
    
    Parent(int n) {
        a = n;
    }
}

class Child extends Parent {
    int b;
    
    Child() {
        super(); // 오류 발생
        b = 20;
    }
}
```
- 부모 클래스 자체에 기본 생성자가 추가되지 않았기 때문에 오류가 발생한다.
- 따라서 (1) 부모 클래스에 기본 생성자를 명시적으로 선언해주거나 (2) 오버로딩된 생성자에 맞춰 super()의 인자를 맞춰줘야 한다.
```java
class Parent {
    int a;
  
    Parent() { a = 10; } // 1
  
    Parent(int n) { a = n; }
}

 

class Child extends Parent {
    int b;

    Child() {
        super(40); // 2
        b = 20;
    }

    void display() {
        System.out.println(a); // 40
        System.out.println(b); // 20
    }
}

public class Main {
    public static void main(String[] args) {
        Child ch = new Child();
        ch.display();
    }
}
```

#### 참고 사이트
- https://velog.io/@rhdmstj17/java.-super%EC%99%80-super-%EC%99%84%EB%B2%BD%ED%95%98%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0
- https://www.tcpschool.com/java/java_inheritance_super