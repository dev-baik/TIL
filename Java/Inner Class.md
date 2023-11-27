# inner class(내부 & 중첩 클래스)

<img src="https://github.com/dev-baik/TIL/assets/96613859/edfd1b39-5e50-413d-8fe8-4e17773d5445" width="50%" height="50%"/>

> 💡 하나의 클래스 내부에 선언된 또 다른 클래스를 의미하며, 두 클래스가 서로 긴밀한 관계가 있거나, 하나의 클래스 또는 메소드에서만 사용되는 클래스일 때 이용되는 기법이다.

```java
// Creature 클래스는 내부 클래스들의 외부 클래스
class Creature {
    int life;
    
    // Animal 클래스는 Creature 클래스의 내부 클래스
    class Animal {}
    
    // Insect 클래스는 Creature 클래스의 내부 클래스
    class Insect {}
    
    public void method() {
        Animal animal = new Animal();
        Insect insect = new Insect();
    }
}
```

## 내부 클래스의 장점

> 💡 클래스를 논리적으로 그룹화하며, 외부 클래스의 모든 멤버에 자유롭게 접근할 수 있고, 외부에서는 내부 클래스의 세부 구현을 감춰서 클래스 간의 상호작용만 고려하는 것과 같은 코드의 복잡성을 줄일 수 있다. 

1. 클래스를 논리적으로 그룹화
- 일반적으로 사용자 클래스 자료형이 필요하면, 메인 클래스 외부에 선언하거나, 따로 독립적인 클래스 파일을 만들어 불러와 사용해 왔다.
```java
class Creature {
    int life;
    
    public void method() {
        // Animal 객체는 오로지 Creature 클래스의 메소드 내에서만 사용된다는 가정
        Animal animal = new Animal();
    }
}

// 외부에 선언된 클래스
class Animal {}
```
- 클래스가 여러 클래스와 관계를 맺지 않고 하나의 특정 클래스와만 관계를 맺는다면, 외부 클래스를 새로 작성하는 것이 아닌 내부 클래스로 작성할 수 있다.
- 내부 클래스와 외부 클래스를 함께 관리하는 것이 가능해 유지보수 면에서나 코드 이해성 면에서 편리해진다.
- 내부 클래스로 인해 새로운 클래스를 생성하지 않아도 되므로 패키지를 간소화할 수 있다.
```java
class Creature {
    int life;
    
    // 클래스 멤버 같이 Creature 클래승 안에다 넣어 선언한다.
    class Animal {}
    
    public void method() {
        Animal animal = new Animal();
    }
}
```

2. 더욱 타이트한 캡슐화의 적용
- 내부 클래스에 private 제어자를 적용해줌으로써, 캡슐화를 통해 클래스를 내부로 숨길 수 있다.
  - 즉, 캡슐화를 통해 외부에서의 접근을 차단하면서도 내부 클래스에서 외부 클래스의 멤버들을 제약 없이 쉽게 접근할 수 있어 구조적인 프로그래밍이 가능해진다. 그리고 클래스 구조를 숨김으로써 코드의 복잡성도 줄일 수 있다.
```java
class Creature {
    private int life = 50;
    
    // private class로 오로지 Creature 외부 클래스에서만 접근 가능한 내부 클래스로 설정
    private class Animal {
        private String name = "호랑이";
        
        int getOuter() {
            // 외부 클래스의 private 멤버를 제약 없이 접근 가능
            return life;
        }
    }
    
    public void method() {
        Animal animal = new Animal();
        
        // Getter 없이 내부 클래스의 private 멤버에 접근이 가능
        System.out.println(animal.name); // 호랑이
        
        // 내부 클래스에서 외부 클래스의 private 멤버를 출력
        System.out.println(animal.getOuter()); // 50
    }

    public static void main(String[] args) {
        Creature creature = new Creature();
        creature.method();
    }
}
```

3. 가독성이 좋고 유지 관리가 쉬운 코드
- 내부 클래스를 작성하는 경우 클래스를 따로 외부에 작성하는 경우보다, 물리적으로 논리적으로 외부 클래스에 더 가깝게 위치하게 된다. 따라서 시각적으로 읽기가 편해질 뿐 아니라 유지보수에 있어 이점을 가지게 된다.
- 한 클래스를 다른 클래스의 내부 클래스로 선언하면 두 클래스 멤버들 간에 서로 자유롭게 접근할 수 있고, 외부에는 불필요한 클래스를 감춰서 클래스 간의 연관 관계를 따지는 것과 같은 코드의 복잡성을 줄일 수 있다는 장점이 있다.

## 내부 클래스 종류
- 클래스 멤버 변수도 선언되는 위치나 접근 제어자에 따라 역할과 이름이 달라지듯이, 내부 클래스도 선언된 위치, static 키워드의 유무 등에 따라 4가지로 내부 클래스가 구분된다.
```java
class Outer {
    class InstanceInner { ... } // 인스턴스 클래스
    static class StaticInner { ... } // static 클래스
    
    void method1(){
    	class LocalInner { ... } // local 클래스
    }
}
```

### 인스턴스 클래스
- 외부 클래스의 멤버 변수 선언 위치에 선언하며, 외부 클래스의 인스턴스 멤버처럼 다뤄진다. 
- 주로 외부 클래스의 인스턴스 멤버들과 관련된 작업에 사용될 목적으로 선언된다.
```java
class PocketBall {
    // 인스턴스 변수
    int size = 100;
    int price = 5000;
  
    // 인스턴스 내부 클래스
    class PocketMonster {
        String name = "이상해씨";
        int level = 10;
    
        // Error: 인스턴스 내부 클래스에는 static 변수를 선언할 수 없다.
        // static int cost = 100;
    
        // final static은 상수이므로 허용
        static final int cost = 100;
    
        public void getPoketMember() {
            // 별다른 조치 없이 외부 클래스 멤버 접근 가능
            System.out.println(size);
            System.out.println(price);
      
            // 내부 클래스 멤버
            System.out.println(name);
            System.out.println(level);
            System.out.println(cost);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        // 1. 외부 클래스를 인스턴스화 해주기
        PocketBall ball = new PocketBall();
        // 2. 외부클래스.내부클래스 형식으로 내부 클래스를 초기화하여 사용할 수도 있다
        PocketBall.PocketMonster pocketmon = ball.new PocketMonster();
        pocketmon.getPoketMember();
    
        // 1 + 2
        PocketBall.PocketMonster poketmon2 = new PocketBall().new PocketMonster();
    }
}
```
- 외부 클래스를 인스턴스화하면 외부 클래스의 코드가 메모리에 올라오게 되고 이 때 내부 클래스의 코드도 메모리에 올라오게 된다. 이렇게 코드를 메모리에 올린 이후에야 내부 클래스의 인스턴스를 생성할 수 있다.

#### 이름이 같은 외부 클래스 메서드 호출
- 내부 클래스에서 외부 클래스와 동일한 메서드명을 사용할 때 외부 클래스의 메서드를 어떻게 호출해야 할까?
```java
// 외부 클래스
public class Main {
    public print(String txt) {
        System.out.println(txt);
    }
    
    // 내부 클래스
    class Sub {
        public print() {}
    }
}
```
- 클래스가 상속 관계에 있을 때는 [super 키워드](https://velog.io/@dev-baik/super-%ED%82%A4%EC%9B%8C%EB%93%9C#super)를 통해 부모 메서드를 호출할 수 있다.
- 하지만 외부-내부 클래스 관계는 상속 관계가 아니기 때문에 정규화된 this를 사용하여 외부 클래스의 멤버를 호출해야 한다.
  - 정규화된 this : 클래스명.this 형태로 바깥 클래스의 이름을 명시하는 용법
```java
public class Main {
    public void print(String txt) {
        System.out.println(txt);
    }
    
    class Sub {
        public void print() {
            Main.this.print("외부 클래스 메소드 호출");
            System.out.println("내부 클래스 메소드 호출");
        }
    }
}

public static void main(String[] args) {
    Main.Sub s = new Main().new Sub();
    s.print();
    // 외부 클래스 메소드 호출
    // 내부 클래스 메소드 호출
}
```

### static 클래스
- 외부 클래스의 멤버 변수 선언 위치에 선언하며, 외부 클래스의 정적 멤버처럼 다뤄진다.
- <span style='background-color: #fff5b1'/>주의할 점 : 일반적인 정적 멤버와 달리, 정적 내부 클래스와 같은 static이지만 메모리 구조나 기능이 전혀 다르다.
```java
public class Main {
    static Integer num = new Integer(0); // 정적 필드 변수

    class InnerClass { } // 내부 인스턴스 클래스
    
    static class InnerStaticClass { } // 내부 정적 클래스
    
    public static void main(String[] args) {
        // 정적 필드 변수는 유일해서 서로 같다.
        Integer num1 = Main.num;
        Integer num2 = Main.num;
        System.out.println(num1 == num2); // true
        
        // 생성된 내부 클래스 인스턴스는 서로 다르다.
        Main.InnerClass inner1 = new Main().new InnerClass();
        Main.InnerClass inner2 = new Main().new InnerClass();
        System.out.println(inner1 == inner2); // false
        
        // 생성된 내부 정적 클래스 인스턴스는 서로 다르다.
        Main.InnerStaticClass static1 = new InnerStaticClass();
        Main.InnerStaticClass static2 = new InnerStaticClass();
        System.out.println(static1 == static2); // false
    }
}
```
- 즉, 정적 클래스 내부에는 인스턴스 멤버와 정적 멤버 모두 선언할 수 있다.
- 단, 일반적인 정적 메서드와 동일하게 외부 클래스의 인스턴스 멤버에는 접근이 불가하고, 정적 멤버에만 접근할 수 있다.
```java
class PocketBall {
    int size = 100;
    static int price = 5000;
    
    // static 내부 클래스
    static class PocketMonster {
        static String name = "이상해씨";
        int level = 10;
        
        public static void getPoketMember() {
            // 외부 클래스 인스턴스 멤버 접근 불가능
            // System.out.println(size);
            
            // 외부 클래스 정적 멤버 접근 가능
            System.out.println(price);
            
            // 내부 클래스 멤버도 정적 멤버만 접근 가능
            // System.out.println(level);
            
            System.out.println(name);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        // 정적 내부 클래스의 인스턴스는 외부 클래스를 먼저 생성하지 않아도 된다.
        PocketBall.PocketMonster poketmon = new PocketBall.PocketMonster();
        System.out.println(poketmon.level);
        System.out.println(PocketBall.PocketMonster.name);
        
        // 클래스.정적내부클래스.정적메소드()
        PocketBall.PocketMonster.getPoketMember();
    }
}
```
- 내부 클래스에서 바깥 외부의 인스턴스를 사용하지 않을 경우 static 클래스로 선언해주어야 한다.
  - static이 아닌 내부 인스턴스 클래스는 외부와 연결되어있어 외부 참조를 갖게된다.

### local(지역) 클래스
- 외부 클래스의 메서드나 초기화 블럭 안에 선언하며, 선언된 메서드 블록 영역 내부에서만 사용될 수 있다.
- 접근제한자와 static을 붙일 수 없다.
  - WHY) 메소드 내부에서만 사용되므로 접근을 제한할 필요가 없고, 원래 메소드 내에는 static 멤버를 선언할 수 없기 때문이다.
```java
class PocketBall {
    int size = 100;
    int price = 5000;
    
    public void pocketMethod() {
        int exp = 5000;
        
        // 메소드 내에서 클래스를 선언
        class PocketMonster {
            String name = "이상해씨";
            int level = 10;
            
            public void getPoketLevel() {
                System.out.println(level); // 인스턴스 변수 출력
                System.out.println(exp); // 메소드 지역 상수 출력
            }
        }
        
        // 메소드 내에서 클래스를 선언
        class PocketMonster2 {
            String name = "리자몽";
            int level = 50;
        }
        
        new PocketMonster().getPoketLevel();
        System.out.println("메소드 실행 완료");
    }
}
```
- 메소드 내에서 인스턴스를 생성한 후 사용하고 메소드 종료와 함께 레퍼런스가 사라지면서 힙 메모리 영역의 실제 데이터 나중에 GC(가비지 컬렉션)에 의해 지워지게 된다.

#### 로컬 클래스 지역 상수 접근
- 메서드 내의 로컬 클래스에서 지역 변수에 접근해서 값을 사용하려고 할 때 반드시 final 상수화된 지역 변수만 사용 가능하다.
```java
public void pocketMethod() {
    int exp = 5000;
    exp = 1;
    
    // 메소드 내에서 클래스를 선언 (final 자동 붙음)
    class PocketMonster {
        /*final*/ String name = "이상해씨";
        /*final*/ int level = 10;
        
        public void getPoketLevel() {
            System.out.println(level); // 인스턴스 변수 출력
            // System.out.println(exp); // 메소드 지역 변수 출력 -> 컴파일 에러
        }
    }
}
```

### 익명 클래스
- 클래스의 선언과 객체의 생성을 동시에 하는 이름 없는 클래스이다.
  - 단 하나의 객체만을 생성하는 일회용 클래스
- 익명 클래스는 기존에 존재하는 클래스를 메서드 내에서 일회용으로 클래스 내부 구성을 선언하여 필요한 메서드를 재정의하여 사용하는 기법이다.
- 생성자를 선언할 수 없고, 오로지 단 하나의 클래스나 단 하나의 인터페이스를 상속받거나 구현할 수 있다.

```java
public class Main {
    public static void main(String[] args) {
        // Object 클래스를 일회성으로 익명 클래스로 선언하여 변수 o에 저장
        Object o = new Object() {
            String t = "안녕";
            
            @Override
            public String toString() {
                System.out.println(this.t);
                  return  "내 마음대로 toString 바꾸기";
            }
        };
    
        // 익명 클래스의 객체의 오버라이딩한 메서드를 사용
        String txt = o.toString();
        System.out.println(txt); // 내 마음대로 toString 바꾸기
    }
}
```

#### 참고한 사이트
- https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4Inner-Class-%EC%9E%A5%EC%A0%90-%EC%A2%85%EB%A5%98