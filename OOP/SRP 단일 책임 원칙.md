# SRP (Single Responsibility Principle)

**WHAT)** 프로그램의 유지보수 성을 높이기 위한 설계 기법으로, 클래스(객체)는 단 하나의 책임(기능)만 가져야 한다는 원칙

즉, 하나의 클래스는 하나의 기능을 담당하여 하나의 책임을 수행하는 데 집중되도록 클래스를 따로따로 여러개 설계하라는 원칙

**WHY)** 하나의 클래스에 기능(책임)이 여러개 있다면 기능 변경(수정)이 일어났을 때 수정해야할 코드가 많아진다.

즉, SRP 원칙을 따름으로써 한 책임의 변경으로부터 다른 책임의 변경으로의 연쇄작용을 극복할 수 있게 된다.

# SRP 원칙 위반예제 1
## 기존 코드
- calculatePay() : 회계팀에서 급여를 계산하는 메서드
- reportHours() : 인사팀에서 근무시간을 계산하는 메서드
- saveDababase() : 기술팀에서 변경된 정보를 DB에 저장하는 메서드
- calculateExtraHour() : 초과 근무 시간을 계산하는 메서드 (회계팀과 인사팀에서 공유하여 사용)

```kotlin
class Employee(val name: String, val position: String) {
    // 초과 근무 시간을 계산하는 메서드 (두 팀에서 공유하여 사용)
    fun calculateExtraHour() { /*...*/ }

    // 급여를 계산하는 메서드 (회계팀에서 사용)
    fun calculatePay() {
        calculateExtraHour()
    }

    // 근무시간을 계산하는 메서드 (인사팀에서 사용)
    fun reportHours() {
        calculateExtraHour()
    }

    // 변경된 정보를 DB에 저장하는 메서드 (기술팀에서 사용)
    fun saveDatabase() { /*...*/ }
}
```

## 요구사항 업데이트
- **문제점** : 개발팀에서 회계팀의 요청에 따라 calculateExtraHour() 메서드를 변경했는 데, 변경에 의한 파급 효과로 인해 수정 내용이 의도치 않게 reportHours() 메서드에도 영향을 주게 되어버린다.
- **WHY)** 회계팀, 인사팀, 기술팀에서 데이터를 얻기위해 하나의 Employee 클래스를 사용하기 때문에, 3개의 액터가 하나의 클래스를 변경할 수 있는 요인이 되다. 
  - 즉, Employee 클래스에서 3개의 엑터(회계팀, 인사팀, 기술팀)에 대한 책임을 한꺼번에 가지고 있기 때문이다

> ✅ **액터** : 시스템을 수행하는 역할을 하는 요소로서, 시스템을 이용하는 사용자, 하드웨어 혹은 외부 시스템이 될 수 있다.

# SRP 원칙 준수예제 1
```kotlin
// 통합 사용 클래스
class EmployeeFacade(private val name: String, private val position: String) {
    // 급여를 계산하는 메서드 (회계팀 클래스를 불러와서 사용)
    fun calculatePay() {
        PayCalculator().calculatePay()
    }

    // 근무시간을 계산하는 메서드 (인사팀 클래스를 불러와서 사용)
    fun reportHours() {
        HourReporter().reportHours()
    }

    // 변경된 정보를 DB에 저장하는 메서드 (기술팀 클래스를 불러와서 사용)
    fun employeeSaver() {
        EmployeeSaver().saveDatabase()
    }
}

// 회계팀에서 사용되는 전용 클래스
class PayCalculator {
    // 초과 근무 시간을 계산하는 메서드
    fun calculateExtraHour() { /*...*/ }

    fun calculatePay() {
        calculateExtraHour()
    }
}

// 인사팀에서 사용되는 전용 클래스
class HourReporter {
    // 초과 근무 시간을 계산하는 메서드
    fun calculateExtraHour() { /*...*/ }

    fun reportHours() {
        calculate, ExtraHour()
    }
}

// 기술팀에서 사용되는 전용 클래스
class EmployeeSaver {
    fun saveDatabase() { /*...*/ }
}
```

### [Facade 패턴](https://ko.wikipedia.org/wiki/%ED%8D%BC%EC%82%AC%EB%93%9C_%ED%8C%A8%ED%84%B4)
- Facade Pattern : 건물의 뒷부분이 어떻게 생겼는지 보여주지 않고 건물의 정면만 보여주는 패턴
- 즉, EmployeeFacade 클래스는 메서드의 구현이 어떻게 되어있는지는(건물의 뒷부분) 보여주지 않고, 어떤 메서드가 있는지(건물의 정면)만 보여준다.
- 구체적인 메서드의 구현은 각각 PayCalculator, HourReporter, EmployeeSaver 클래스에 위임하기 때문이다.

### 캡슐화(정보 은닉)
- 클래스를 사용하는 입장에선 PayCalculator, HourReport, EmployeeSaver 클래스가 어떤식으로 구성되어있는지 알필요 없이 EmployeeFacade 클래스를 불러와 메서드만 사용하면 되기 때문에 캡슐화(정보 은닉)도 잘 구성되어있다

# SRP 원칙 위반예제 2
## 기존 코드
- EmployeeManagement : 직원에 대한 기본 CRUD 작업을 담당하는 클래스
```kotlin
class EmployeeManagement {
    // Create 작업을 담당하는 CRUD 메소드
    fun addEmployee(employee: String) {
        if (employee == "") {
            postServer(employee) // 서버에 직원 정보를 보냄
            logResult("[LOG] EMPLOYEE ADDED") // 로그 출력
        } else {
            logResult("[ERROR] NAME MUST NOT BE EMPTY")
        }
    }

    // 서버에 데이터를 전달하는 메소드
    fun postServer(employees: String) {}

    // 로그를 출력하는 메소드
    fun logResult(message: String) {
        println(message) // 로그를 콘솔에 출력
        writeToFile(message) // 로그 내용을 로그 파일에 저장
    }

    // 파일에 내용을 저장하는 메소드
    fun writeToFile(msg: String) {}
} 
```
- **문제점** : 서버에 데이터를 보내는 동작과 작업 결과를 파일에 기록하는 동작은 서로 관련된 동작이 아니다.
- **HOW)** 로깅만을 담당하는 클래스를 따로 분리하고, EmployeeManagement 클래스에서 합성하여 사용한다.

# SRP 원칙 준수예제 2
```kotlin
class EmployeeManagement {
    private val logger = Logger() // 합성

    // Create 작업을 담당하는 CRUD 메소드
    fun addEmployee(employee: String) {
        if (employee == "") {
            postServer(employee) // 서버에 직원 정보를 보냄
            logger.logResult("[LOG] EMPLOYEE ADDED") // 로그 출력
        } else {
            logger.logResult("[ERROR] NAME MUST NOT BE EMPTY")
        }
    }

    // 서버에 데이터를 전달하는 메소드
    fun postServer(employees: String) {}
}

class Logger {
    // 로그를 출력하는 메소드
    fun logResult(message: String) {
        println(message) // 로그를 콘솔에 출력
        writeToFile(message) // 로그 내용을 로그 파일에 저장
    }

    // 파일에 내용을 저장하는 메소드
    fun writeToFile(msg: String) {}
}
```

> 어떤 프로그램을 개발하느냐에 따라 개발자의 생각이 제각기 다르기 때문에 단일 책임 원칙에 100% 답은 없다. 하지만 중요한 것은 클래스를 작성할 때 단일 책임 원칙을 지켰는지 끊임 없이 생각해보는 것이다. 하나의 클래스가 너무 많은 책임을 가지진 않았는지, 분리할 수 있는 변수와 메소드가 많은 것은 아닌지를 항상 고민해 봐야 한다.

# SRP 원칙 적용 주의점
## 1. 클래스명은 책임의 소재를 알 수 있게 작명
클래스가 하나의 책임을 가지고 있다는 것을 나타내기 위해, 클래스명으로 어떠한 기능을 담당하는지 알 수 있게 작명하는 것이 좋다.

## 2. 책임을 분리할 때 항상 결합도, 응집도 따져가며 구성해야 한다.
응집도 : 한 프로그램 요소가 얼마나 뭉쳐있는가를 나타내는 척도

결합도 : 프로그램 구성 요소들 사이가 얼마나 의존적인지를 나타내는 척도

좋은 프로그램이란 응집도를 높게, 결합도는 낮게 설계하는 것

따라서 여러가지 책임으로 나눌 때는 각 책임 간의 결합도를 최소로 하도록 코드를 구성해야 한다.

하지만 그 반대로 너무 많은 책임 분할로 인하여 책임이 여러군데로 파편화되어있는 경우에는 산탄총 수술로 다시 응집력을 높여주는 작업이 추가로 필요하다.

### 산탄총 수술
- 반대로 하나의 책임 담당이여러 개의 클래스들로 분산되어 있는 경우에도, 단일 책임 원칙에 입각해 설계를 변경해야 하는 케이스도 존재한다.
- 예를 들어 로깅, 보안, 트랜잭션과 같은 시스템 안에 포함되는 부가 기능을 이것도 하나의 책임으로 보고 분리하는 것이다.

<img src="https://velog.velcdn.com/images/dev-baik/post/b186ce4e-c5af-4258-99cb-69e20282e640/image.png"/>

- 여러개의 모듈에서 자체적으로 로깅, 보안, 트랜잭션을 처리하고 있는데, 아무리 로깅과 같은 사소하고 부가적인 기능이라도 여러 모듈에 공통적으로 자주 이용된다면 책임 소지를 분리해 따로 클래스로 관리하라는 뜻이다.

#### 참고한 사이트
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-SRP-%EB%8B%A8%EC%9D%BC-%EC%B1%85%EC%9E%84-%EC%9B%90%EC%B9%99