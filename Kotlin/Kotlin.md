# Kotlin
자바와의 원활한 상호 운용성을 제공하며, 정적 타입 지정 언어로서 널 안전성과 함수형 프로그래밍을 지원하여 코드의 간결성과 안전성을 높이는 대체 언어

- - -

## 상호 운용성
자바 메서드를 호출하거나, 자바 클래스를 상속하거나, 인터페이스를 구현하거나, 자바 애노테이션을 코틀린 코드에 적용하는 등 기존 자바 라이브러리를 그대로 사용할 수 있는 것

- - -

## 정적 타입 지정
**개념** : 모든 프로그램 구성 요소의 타입을 컴파일 시점에 알 수 있고, 프로그램 안에서 객체의 필드나 메서드를 사용할 때마다 컴파일러가 타입을 검증해준다.

**장점** : 실행 시점에 어떤 메서드를 호출할지 알아내는 과정이 필요없으므로 메서드 호출이 더 빠르고, 컴파일러가 프로그램의 정확성을 검증하기 때문에 실행 시 프로그램이 오류로 중단될 가능성이 더 작아진다.

### 컴파일 시점
컴파일러를 통해 개발자가 작성한 소스 코드를 시스템이 이해할 수 있는 기계어로 번역하는 데 발생하는 시간으로, 소스 코드를 수정하거나 추가할 때 오류를 미리 발견하여 안전성을 높이는 역할을 한다.

#### 컴파일러
소스 코드를 컴퓨터가 이해하고 실행할 수 있는 기계어로 변환하는 소프트웨어 도구

#### 타입 추론
컴파일러가 문맥으로부터 변수 타입을 자동으로 유추해주는 것으로, 타입 선언을 생략해도 된다.

## vs 동적 타입 지정
타입과 관계없이 모든 값을 변수에 넣을 수 있고, 메서드나 필드 접근에 대한 검증이 런타임에 일어난다. 따라서 이름을 잘못 입력하는 등의 실수도 컴파일 시 걸러내지 못하고 실행 시점에 오류가 발생한다.

### 런타임 시점
컴파일된 프로그램이 실제로 실행되는 시점으로, 프로그램이 실행되는 동안 발생하는 시간을 의미한다. 이 동안에 프로그램은 작업을 수행하고 데이터를 처리하며 사용자와 상호작용한다.

- - -

## 널 안전성
널이 될 수 있는 타입을 지원함에 따라 컴파일 시점에 널 포인터 예외가 발생할 수 있는지 여부를 검사할 수 있어서 좀 더 프로그램의 신뢰성을 높일 수 있다.

> **Q1. 좀 더 프로그램의 신뢰성을 높일 수 있다면?** : 런타임 중에 예기치 않는 널 포인터 예외를 발생시키지 않게 된다.. 따라서, 프로그램이 예외로 중단되는 상황을 방지하고 안정성을 확보할 수 있다.

- - -

## 함수형 프로그래밍
문제를 작은 단위로 분할하고, 각각의 문제를 해결하기 위한 순수 함수를 사용하여 부수효과를 최소화하는 프로그래밍 패러다임이다. 함수를 1급 객체로 다루고, 참조 투명성을 유지하여 예측 가능하고 안정적인 코드를 작성한다.

### 순수 함수 (<span style='background-color: #F7DDBE'>참조 투명성</span>, <span style='background-color: #fff5b1'>불변성</span>)
**개념** : <span style='background-color: #F7DDBE'>동일한 입력에 대해 항상 동일한 출력을 반환하고</span>, <span style='background-color: #fff5b1'>함수나 표현식을 실행하더라도 외부 객체나 상태를 변경하지 않으며</span>, <span style='background-color: #F7DDBE'>외부 환경과 상호작용하지 않는</span> 함수이다. 함수 자체가 독립적이며 부수효과가 없기 때문에 병렬 처리를 동기화 없이 안전하게 작동할 수 있다.

**장점** :
1. 순수 함수를 값처럼 활용하면 더 강력한 추상화를 할 수 있고 강력한 추상화를 사용해 코드 중복을 막을 수 있다.
2. 순수 함수를 사용하면 불변 데이터 구조를 적용함으로써 코드를 더 예측 가능하고 테스트하기 쉽다.

> **Q2. 부수 효과가 있다면?** : 부수 효과가 있는 함수는 그 함수를 실행할 때 필요한 전체 환경을 구성하는 준비 코드가 따로 필요하다. 하지만 순수 함수는 그런 준비 코드 없이 독립적으로 테스트할 수 있다.

#### 병렬 처리
<img src="https://velog.velcdn.com/images/dev-baik/post/36e40c87-2e19-49c8-93ec-5f7183609e25/image.png" width="100%"/>

컴퓨터 프로그램이 동시에 여러 작업을 처리하는 것을 의미하며, 데이터 공유로 인한 동시 업데이트 문제가 발생할 수 있으므로 동기화 기술을 사용하여 안전하게 데이터를 공유해야 한다.

**동시성(Concurrent)**

<img src="https://velog.velcdn.com/images/dev-baik/post/14a7a435-1e85-41a0-9e03-4bc6a46f4862/image.png" width="100%"/>

싱글 코어에서 멀티 스레드를 동작시키기 위한 방식으로, 멀티 태스킹을 위해 여러 개의 스레드가 번갈아가면서 실행되고, 각 스레드들이 병렬적으로 실행되는 것처럼 보이지만, 번갈아가면서 조금씩 실행된다.

- **멀티 태스킹** : 하나의 시스템에서 여러 개의 작업을 동시에 실행하는 것을 의미하며, 운영 체제에서는 이를 위해 여러 개의 프로세스 (또는 스레드)를 생성하여 동시에 실행합니다. 이러한 멀티태스킹 방식은 하나의 CPU를 사용하여 여러 작업을 처리하므로 시스템 자원의 효율성이 높아진다.

- **동시성** : 하나의 작업 내에서 여러 개의 서브 태스크를 동시에 처리하는 것을 의미하며, 여러 개의 스레드를 생성하여 하나의 작업을 분할하여 처리하거나, 비동기적으로 여러 개의 작업을 처리하는 것 등을 말한다. 동시성은 멀티태스킹과 마찬가지로 동시에 작업을 수행하지만, 작업이 독립적이지 않고 서로 의존성을 가지는 경우에 적합하다.
```kotlin
import kotlinx.coroutines.*

suspend fun doTask1(): String {
    delay(1000) // 1초 동안 대기
    return "Task 1 완료"
}

suspend fun doTask2(): String {
    delay(2000) // 2초 동안 대기
    return "Task 2 완료"
}

suspend fun doTask3(): String {
    delay(3000)
    return "Task 3 완료"
}

fun main() = runBlocking {
    val job1 = launch {
        val result = doTask1()
        println(result)
    }

    val job2 = launch {
        val result = doTask2()
        println(result)
    }

    val job3 = launch {
        val result = doTask3()
        println(result)
    }
    
    job1.join()
    job3.join()
    job2.join()

    println("모든 작업 완료")
}

// Task 1 완료
// Task 2 완료
// Task 3 완료
// 모든 작업 완료 
```

**병렬성(Parallel)**

<img src="https://velog.velcdn.com/images/dev-baik/post/e98644fa-a7b5-4668-a9eb-978630f64bd4/image.png" width="100%"/>

멀티 코어에서 멀티 스레드를 동작시키는 방식으로, 한 개 이상의 스레드를 포함하는 각 코어들이 동시에 실행되는 것을 의미하며, 데이터 병렬성과 작업 병렬성으로 구분된다.

- **데이터 병렬성** : 데이터를 여러 코어로 분할하여 병렬로 처리하는 방식을 의미하며, 대량의 데이터를 동시에 처리할 때 유용하다.

- **작업 병렬성** : 서로 다른 작업을 병렬로 실행하는 방식으로, 각 코어는 서로 다른 작업을 처리하고, 여러 작업이 동시에 실행된다. 주로 복잡한 응용 프로그램을 효율적으로 분배하고 처리할 때 유용하다.

```kotlin
import kotlinx.coroutines.async
import kotlinx.coroutines.runBlocking

suspend fun main() = runBlocking {
    val job1 = async {
        for (i in 1..5) {
            println("작업 1: $i")
            kotlinx.coroutines.delay(1000)
        }
    }

    val job2 = async {
        for (i in 1..5) {
            println("작업 2: $i")
            kotlinx.coroutines.delay(1000)
        }
    }

    // 모든 작업이 완료될 때까지 대기
    job1.await()
    job2.await()

    println("모든 작업 완료")
}

// 작업 1: 1
// 작업 2: 1
// 작업 1: 2
// 작업 2: 2
// 작업 1: 3
// 작업 2: 3
// 작업 1: 4
// 작업 2: 4
// 작업 1: 5
// 작업 2: 5
// 모든 작업 완료
```

#### 동기화
여러 쓰레드에서 같은 데이터에 동시에 접근하는 경우, 발생하는 경쟁 상황을 방지하고, 데이터의 일관성을 유지하기 위해 사용되는 것

- **Handler와 MessageQueue** : UI 스레드와 백그라운드 스레드 간의 통신 및 동기화를 위해 사용되며, 메시지 전달을 통해 데이터를 안전하게 교환하고 UI 업데이트를 수행할 수 있다.
```kotlin
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import android.os.Message
import androidx.appcompat.app.AppCompatActivity

class MyActivity : AppCompatActivity() {

    // UI 스레드에서 실행되는 Handler
    private val uiHandler = Handler(Looper.getMainLooper())

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 백그라운드 스레드에서 메시지를 UI 스레드로 전송
        val backgroundThread = Thread {
            // 백그라운드 스레드에서 작업 수행
            // ...

            // UI 스레드로 메시지 보내기
            val message = Message.obtain()
            message.obj = "작업이 완료되었습니다."
            uiHandler.sendMessage(message)
        }
        backgroundThread.start()
    }

    // UI 스레드에서 메시지 처리
    private val messageHandler = object : Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message) {
            val result = msg.obj as String
            // UI 스레드에서 작업 결과 처리
            // 예를 들어, 텍스트뷰에 결과 출력
        }
    }
} 
```

- **Thread 및 Runnable** : 직접 스레드를 만들고 관리하며, 스레드 간의 데이터 공유 및 동기화는 적절한 메커니즘을 사용하여 수행된다.
```kotlin
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import androidx.appcompat.app.AppCompatActivity

class MyActivity : AppCompatActivity() {

    // UI 스레드에서 실행되는 Handler
    private val uiHandler = Handler(Looper.getMainLooper())

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 백그라운드 스레드 시작
        val backgroundThread = Thread(Runnable {
            // 백그라운드 스레드에서 비동기 작업 수행
            val result = performBackgroundTask()

            // UI 스레드로 결과 전달
            uiHandler.post {
                handleResultOnUIThread(result)
            }
        })

        backgroundThread.start()
    }

    private fun performBackgroundTask(): String {
        // 백그라운드 스레드에서 비동기 작업 수행
        // 예를 들어, 네트워크 요청, 데이터 처리 등
        return "작업이 완료되었습니다."
    }

    private fun handleResultOnUIThread(result: String) {
        // UI 스레드에서 작업 결과 처리
        // 예를 들어, 텍스트뷰에 결과 출력
    }
} 
```

### 프로그래밍 패러다임
프로그래머에게 코드 작성 방법과 문제 해결 방법에 대한 접근 방식을 안내한다. 각 프로그래밍 패러다임은 고유한 스타일과 원칙을 가지며, 프로그래머가 특정 문제를 어떻게 해결할지 결정하는 데 도움을 준다.

- **명령형 프로그래밍** : 무엇(WHAT)을 할 것인지 나타내기보다 어떻게(HOW) 할 것인지를 설명하는 방식
    - **절차지향 프로그래밍** : 수행되어야 할 순차적인 처리 과정을 포함하는 방식 (C, C++)
    - **객체지향 프로그래밍** : 객체들의 집합으로 프로그램의 상호 작용을 표현 (C++, Java, C#)


- **선언형 프로그래밍** : 어떻게(HOW) 할것인지를 나타내기보다 무엇(WHAT)을 할 것인지를 설명하는 방식
    - **함수형 프로그래밍** : 순수 함수를 조합하고 소프트웨어를 만드는 방식

### 부수효과
함수나 작업 실행 중에 변수의 값이 변경되거나 객체의 필드값이 설정되는 등의 작업을 나타내며, 종종 예외나 오류가 발생하여 실행이 중단되는 상황을 의미한다.

### 1급 함수
함수를 일반 값처럼 다룰 수 있어 함수를 변수에 저장하거나 인자로 다른 함수에 전달할 수 있고, 새로운 함수를 만들어서 반환할 수 있다.

### 참조 투명성
동일한 입력에 대해 항상 동일한 출력을 가지고, 함수나 표현식을 실행하더라도 외부 객체나 상태를 변경하지 않으며, 외부 환경과 상호작용하지 않는 특성을 나타낸다.

- - -

## 간결성
코틀린의 간결성은 프로그램을 더 간결하고 효율적으로 작성할 수 있도록 부수적인 요소를 줄이는 방식으로 제공된다. 이는 게터와 세터를 자동으로 생성하거나 생성자 파라미터를 필드에 대입하기 위한 로직을 간소화하여 프로그래머의 생산성을 높이는 데 중점을 둔다.

- - -

## 안전성
코틀린은 정적 타입 지정 언어로서 컴파일 시점에서 타입 검사를 수행하여 오류를 미리 방지하고, null 안전성을 제공하여 null이 될 수 없는 값을 추적한다. 또한, 실행 시점에 널 포인터 예외를 방지하기 위해 null 안전성을 강화하며, 캐스트 예외를 방지하기 위해 스마트 캐스트와 같은 기능을 제공한다. 이를 통해 코드의 안전성을 높이고 오류를 사전에 방지한다.

- - -

#### 참고 사이트
- https://velog.io/@kwontae1313/%EB%8F%99%EC%8B%9C%EC%84%B1%EA%B3%BC-%EB%B3%91%EB%A0%AC%EC%84%B1%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%B0%A8%EC%9D%B4
- https://velog.io/@haero_kim/Thread-vs-Coroutine-%EB%B9%84%EA%B5%90%ED%95%B4%EB%B3%B4%EA%B8%B0