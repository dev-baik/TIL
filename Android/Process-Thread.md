# Process와 Thread의 차이점
- 둘 다 프로그램의 실행과 관련된 단어들
    - **프로그램(Program)** : 어떤 작업을 위해 실행할 수 있는 파일
        - **프로그램과 프로세스의 연관성** : 모든 프로그램은 운영체제가 메모리 공간을 할당해 줘야 실행될 수 있다. 프로그램을 실행하는 순간 파일은 메모리에 올라가게 되고, 운영체제로부터 시스템 자원을 할당받아 프로그램 코드를 실행시킨다.

## Process
- 운영체제로부터 시스템 자원을 할당받는 작업의 단위입니다. 각 프로세스는 독립적인 메모리 공간을 할당받고, 하나 이상의 스레드를 가지며 운영체제에 의해 개별적으로 관리됩니다.
    - **운영체제** : 컴퓨터 시스템의 핵심 소프트웨어로, 컴퓨터 하드웨어와 응용 프로그램 간의 상호작용을 관리하고 제어하는 것
    - **독립적인 메모리 공간**
        - **Code** : 프로그래머가 작성한 프로그램 함수들의 코드가 CPU가 해석 가능한 기계어 형태로 저장되어 있다.
        - **Data** : 코드가 실행되면서 사용하는 전역 변수나 각종 데이터들이 모여있다.
        - **Stack** : 지역 변수와 같은 호출한 함수가 종료되면 되돌아올 임시적인 자료를 저장하고 있다.
        - **Heap** : 생성자, 인스턴스와 같은 동적으로 할당되는 데이터이 저장되어 있다.
    - 하나 이상의 스레드 = <span style='background-color: #fff5b1'/>NEXT) 멀티 스레드

## Thread
- 프로세스가 할당받은 자원을 이용하는 실행 흐름의 단위입니다. 같은 프로세스 내에서 메모리를 공유하여 스레드 간의 데이터 공유가 가능하고, 스레드들은 프로세스 내의 자원을 동시에 수행하거나 병렬로 실행할 수 있습니다.
    - **메모리 공유** : 프로세스 내에서 Stack만 따로 할당 받고 Code, Data, Heap 영역은 공유한다.
    - <span style='background-color: #fff5b1'/>NEXT) 동시성 & 병렬성

---

# Android Thread
## 일반적인 메인 스레드
- 프로세스가 시작될 때 최초의 실행 시작점이 되는 main() 함수로부터 순차적으로 진행되는 하나의 스레드

## 안드로이드의 메인 스레드
- UI 관련 작업을 처리합니다. UI를 생성, 업데이트하거나 사용자의 입력을 처리하는 모든 작업을 포함합니다. 또한, 안드로이드 애플리케이션의 구성 요소(액티비티, 서비스 등) 생명주기 콜백을 실행시킵니다.

### 안드로이드 메인스레드의 UI 작업은 단일 스레드 모델!!
- UI 작업에 있어 경쟁 상태, 교착 상태를 방지하고자, 메인 스레드엔 단일 스레드 모델이 적용된다.
    - 단일 스레드 모델은 자원 접근에 대한 동기화를 신경쓰지 않아도 되고, 작업 전환 비용을 요구하지 않으므로, 경쟁 상태와 교착 상태를 방지할 수 있다.
        - **프로세스/스레드 컨텍스트 스위칭(작업 전환)** : CPU/코어에서 실행중이던 프로세스/스레드가 다른 프로세스/스레드로 교체되는 것
            - **자세히** : 동작 중인 프로세스가 대기를 하면서 해당 프로세스의 상태(Context)를 보관하고, 대기하고 있던 다음 순서의 프로세스가 동작하면서 이전에 보관했던 프로세스의 상태를 복구하는 작업을 말한다.
                - **멀티 프로세스**
                    - **장점** : 각 프로세스가 독립적인 메모리 공간을 가지므로, 한 프로세스가 비정상적으로 종료되어도 다른 프로세스에 영향을 주지 않아 프로그램 전체의 안전성을 확보할 수 있다.
                    - **단점** : 프로세스를 컨텍스트 스위칭하면, CPU는 다음 프로세스의 정보를 불러오기 위해 메모리를 검색하고, CPU 캐시 메모리를 초기화하며, 프로세스 상태를 저장하고, 불러올 데이터를 준비해야 하기 때문에, 빈번한 Context Switching 작업은 많은 비용을 발생시킨다.
                - **멀티 스레드**
                    - **장점** : 프로세스 내의 스레드들은 code, data, heap 영역 메모리를 공유하기 때문에 TCB에는 stack과 간단한 레지스터 값만을 저장하고, PCB보다 가벼워 더 빨리 읽고 쓸 수 있다.
                        - **PCB(Process Control Block)** : 운영체제에서 프로세스를 관리하기 위해 해당 프로세스의 상태 정보를 담고 있는 자료구조
                        - **TCB(Thread Control Block)** : 각 스레드마다 운영 체제에서 유지하는 스레드에 대한 정보를 담고 있는 자료구조
                        - **레지스터** : 현재 프로세스가 요청을 처리하는데 필요한 데이터를 일시적으로 저장하는 기억장치
                    - **단점** : 여러 개의 스레드가 공유 자원에 동시에 접근하면 데이터의 일관성과 무결성이 손상될 수 있다. 따라서, 스레드 간의 동기화를 적절히 관리하여 경쟁 상태, 교착 상태 등의 문제를 방지해야 한다.

    - 하나의 위젯에 멀티 스레드를 사용한다면, 동일 자원에 대한 경쟁 상태, 교착 상태 등의 문제가 발생될 수 있다. 더불어 여러 개의 위젯를 사용한다면, 각 위젯이 그려지거나 업데이트되는 순서를 보장할 수 없다.
        - **경쟁 상태** : 공유 자원에 여러 프로세스가 동시에 접근을 시도할 때, 접근의 타이밍이나 순서 등이 결과값에 영향을 줄 수 있는 상태
        - **교착 상태(deadlock)** : 두 개 이상의 작업이 서로 상대방의 작업이 끝나기만을 기다리고 있기 때문에 아무것도 완료되지 못하는 상태
        - <span style='background-color: #fff5b1'/>TODO([경쟁 & 교착 상태에 대한 발생 조건 및 해결 방법](https://velog.io/@kku64r/deadlock))


### [ANR](https://developer.android.com/topic/performance/vitals/anr?hl=ko)

- WHAT) 사용자가 앱을 실행 중일 때, UI 스레드가 너무 오랫동안 차단되어 앱이 일정 시간 내에 응답하지 않아 시스템이 강제 종료하거나 다시 시작하는 상황을 나타냅니다.
- WHEN)
    1. 앱이 입력 이벤트(예: 키 누름 또는 화면 터치)에 5초 이내에 응답하지 않은 경우
    2. Service, BroadcastReceiver, JobScheduler → <span style='background-color: #fff5b1'/>TODO(상황별 해결 방법)
- HOW)
    - 메인 스레드에서 블로킹 작업을 처리하지 않고, 오랜 시간이 걸리는 작업은 백그라운드 스레드에서 처리한다.

## 백그라운드 스레드
- 메인 스레드와는 별개로 실행되며, 파일 I/O, 네트워크 통신, 데이터베이스 접근과 같은 오랜 시간이 걸리는 작업을 처리하는 데 사용된다.
    - **메인 스레드와는 별개로 실행** : UI 자원에 메인 스레드와 서브 스레드가 동시 접근하여 동기화 문제를 발생시키는 것을 방지하기위해 UI 작업은 메인 스레드에서만 가능하도록 만들었다.

---

# 백그라운드 스레드 생성 방법
1. Thread 클래스를 상속받은 서브 클래스를 만들어서 run() 메서드를 구현한 후, 이 클래스의 인스턴스를 생성하고 start() 메서드를 호출하는 것

- **단점** : 스레드 관리나 작업의 동기화를 직접 처리해야 한다.
    - HOW 1) 스레드의 시작, 실행, 종료 등 생명주기를 적절히 관리해야 한다. 스레드가 제대로 종료되지 않으면 메모리 누수나 다른 자원 관리 문제를 일으킬 수 있다.
    - HOW 2) 여러 스레드가 동시에 실행될 때, 공유 자원에 대한 접근을 제어하지 않으면 데이터의 일관성과 무결성이 손상될 수 있다. 따라서, 스레드 간의 동기화를 적절히 관리하여 경쟁 상태, 교착 상태 등의 문제를 방지해야 한다.

- **해결책** :
    - synchronized 블록 사용

    ```kotlin
    var sharedResource: Int = 0
    
    fun synchronizedAccess() {
        val thread1 = Thread {
            synchronized(this) {
                sharedResource++
            }
        }
    
        val thread2 = Thread {
            synchronized(this) {
                sharedResource++
            }
        }
    
        thread1.start()
        thread2.start()
    }
    ```

    - thread1이 먼저 synchronized 블록에 진입하여 작업을 수행하고 있을 때, thread2는 thread1이 블록을 빠져나와 this에 대한 락을 해제할 때까지 대기한다.

    ```kotlin
    companion object {
        private var INSTANCE: AppDatabase? = null
        
        fun getInstance(context: Context): AppDatabase? {
            if(INSTANCE == null) {
                synchronized(AppDatabase::class.java) {
                    INSTANCE = Room.databaseBuilder(
                        context.applicationContext,
                        AppDatabase::class.java,
                        "app-database.db"
                    ).build()
                }
            }
            return INSTANCE
        }
    }
    ```

    - **synchronized(lock: Any, block: () -> R)** : R : 어떤 스레드가 리소스를 사용하는 동안 다른 스레드가 해당 리소스에 접근하지 못하게 하는 락을 제공한다. 따라서 synchronized 블록에서는 단일 스레드만이 실행되기 때문에 다른 스레드는 대기해야 합니다.
        - **lock** : 공유 자원에 대한 접근을 제한하기 위해 사용되는 도구이다. 특정 시점에 하나의 스레드만이 공유 자원에 접근할 수 있도록 하며, 다른 스레드들은 그 자원이 해제될 때까지 대기해야 한다.

- **단점** : 동시 접근으로 인한 경쟁 상태나 교착 상태와 같은 현상은 일어나지 않지만, 여러 스레드 접근을 제한하는 것이기 때문에 병목 현상이 일어나 성능이 저하될 수 있다.
    - **병목 현상** : 시스템 내에서 처리량이나 성능의 한계로 인해 전체 시스템의 성능이나 처리량이 제한되는 현상

      ⇒ NEXT) 병목 현상을 해결할 수 있는 코루틴


2. Runnable 인터페이스를 구현하는 클래스를 만들어서 run() 메서드를 구현한 후, 이 클래스의 인스턴스를 Thread 클래스 인스턴스의 생성자에 전달하고 Thread 클래스 인스턴스의 start() 메서드를 호출하는 것

```kotlin
private fun updateAddWord() {
    Thread {
        AppDatabase.getInstance(this)?.wordDao()?.getLatestWord()?.let { word ->
            wordAdapter.list.add(0, word)
            runOnUiThread { 
                wordAdapter.notifyDataSetChanged() 
            }
        }
    }.start()
}
```

- **public Thread(Runnable target)** : 새로운 Thread 객체를 할당한다.
- **Runnable** : Runnable 인터페이스를 구현하는 객체로 스레드를 생성하면, 스레드를 시작할 때 객체의 run 메서드가 별개의 스레드에서 실행된다.
- **Activity.runOnUiThread** : Runnable 객체를 메인 스레드에서 실행할 수 있도록 도와주는 메서드로, 현재 스레드가 메인 스레드이면 Runnable 객체의 run() 메서드를 직접 실행하고, 메인 스레드가 아닌 경우 Handler에게 post() 메서드를 통하여 메인 스레드로 이벤트 큐를 발송한다.
  - <span style='background-color: #fff5b1'/>NEXT) Handler와 Looper


- **Thread와 Runnable의 차이점** : Thread 클래스는 상속을 통해 새로운 기능을 추가하여 클래스를 확장하거나 태스크의 세부적인 기능을 수정할 수 있고, Runnable 인터페이스는 run() 메서드만을 구현하여 논리적으로 분리된 태스크 설계가 가능하다.


3. Handler와 Looper

<img src="https://velog.velcdn.com/images/dev-baik/post/eb78590a-1898-4d50-9ac6-e4ae048a99eb/image.png"/>

- **메시지(Message)** : 사용자 입력을 포함한 시스템의 모든 이벤트를 전달하기 위해 사용되는 객체로, 핸들러를 통해 메시지 큐에 전송된다.
    - 하나의 데이터를 보내기 위해서는 한 개의 Message 인스턴스가 필요하며, 일단 데이터를 담은 Message 객체를 핸들러로 보내면 해당 객체는 핸들러와 연결된 Message Queue에 쌓이게 된다.
    - Message 객체는 Bundle과 Runnable 객체로 이루어져있다.
- **메시지 큐(MessageQueue)** : Message 객체를 선입선출 방식으로 관리하는 자료구조로, Looper에 의해 관리되며, Handler를 통해 메시지를 전달하고 처리한다.
- **루퍼(Looper)** : 무한 루프를 실행하여 메시지 큐를 모니터링하고, 새로운 메시지가 도착하면 해당 메시지를 처리할 핸들러를 호출하는 역할을 한다.
    - **Looper 객체가 메시지 큐에서 메시지를 하나 꺼냈을 때,**
        1. **Runnable 객체가 담겨있을 경우** : Handler에 메시지를 전달하지 않고 run()을 수행하여 해당 Runnable 작업을 바로 시작한다.
        2. **Runnable 객체가 없을 경우(Message 객체가 담겨있을 경우)** : Message 객체 내부에 명시되어있는 Handler의 handleMessage()를 수행하여 처리한다.
- **핸들러(Handler)** : 다른 스레드로부터 메시지를 받아 Looper와 연결된 메시지 큐로 메시지를 보내거나 handleMessage 메서드를 통해 메시지 처리 로직을 구현하여 메시지를 처리하는 역할을 한다.
    - **Message Queue로 메시지를 전달하는 경우,**
        1. **sendMessage() 메소드** : 메세지 큐에 Message 객체를 적재할 수 있다.
        2. **post로 시작하는 메소드** : Runnable 객체를 직접 적재할 수 있다.

- **Queue와 Stack의 차이점 :** 안드로이드에서 Queue는 선입선출 방식으로 주로 메시지를 관리하거나 Handler와 함께 사용되어 UI 이벤트와 작업 처리 등에 사용된다. 그리고 Stack은 후입선출 방식으로 주로 함수 호출에 대한 정보를 저장하고, 호출 순서에 따라 스택에서 처리된다.

- **runOnUiThread와 Handler의 차이점** : Handler는 post 방식을 통해 매번 이벤트를 발생시키지만, runOnUiThread는 현재 시점이 UI 스레드이면 바로 실행시켜서 좀 더 효율적이라고 할 수 있다.

- 백그라운드 스레드에서 처리해야 할 작업을 마친 후 UI를 갱신해야 하는 경우, 스레드 간 통신을 위해 백그라운드 스레드에서 메인 스레드(UI 스레드)로 처리 결과를 전달해야한다.
    - 파일 I/O, 네트워크 통신, 데이터베이스 접근과 같은 오랜 시간이 걸리는 작업
    1. **백그라운드 스레드 생성** : 백그라운드 스레드를 생성하여 비동기 작업을 수행합니다. 이 작업은 UI 스레드에서 수행하면 안 되는 작업이며, 일반적으로 네트워크 요청이나 파일 I/O 작업 등이 해당됩니다.
    2. **Handler 생성** : 백그라운드 스레드에서 UI 스레드로 메시지를 전달하기 위해 Handler를 생성합니다. Handler는 UI 스레드의 Looper와 연결되어 메시지를 전달할 수 있습니다.
    3. **sendMessage() 또는 post() 메소드 호출** : 백그라운드 스레드에서 생성한 Handler를 사용하여 sendMessage() 또는 post() 메소드를 호출하여 UI 스레드의 Looper의 MessageQueue에 메시지를 전달합니다. 이 메시지에는 UI를 업데이트하는 데 필요한 정보가 포함될 수 있습니다.
    4. **UI 스레드에서의 메시지 처리** : UI 스레드의 Looper는 MessageQueue에서 메시지를 하나씩 꺼내서 해당 메시지를 받을 수 있는 Handler에게 전달합니다.
    5. **Handler의 handleMessage() 호출** : UI 스레드에서 메시지를 받을 수 있는 Handler는 handleMessage() 메서드를 호출하여 해당 메시지를 처리합니다. 이때 UI 업데이트를 포함한 필요한 작업을 수행합니다.

```kotlin
class MainActivity : AppCompatActivity() {

    // 메인 스레드의 Handler 인스턴스를 생성합니다. 이 Handler는 메인 스레드의 Looper와 연결됩니다.
    private val mainHandler = object : Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message) {
            // 메인 스레드에서 메시지를 받아 처리하는 로직을 구현합니다.
            // 예를 들어, TextView의 텍스트를 변경합니다.
            if (msg.what == 1) {
                val text = msg.data.getString("key")
                textView.text = text
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 백그라운드 스레드에서 작업을 시작합니다.
        Thread {
            // 백그라운드 작업을 수행합니다.
            // 예를 들어, 여기서는 단순히 스레드를 잠시 멈춥니다.
            Thread.sleep(2000) // 2초 동안 스레드를 잠시 멈춥니다.

            // Message 객체를 생성하고 데이터를 담습니다.
            val msg = Message.obtain().apply {
                what = 1 // 메시지 식별자
                data = Bundle().apply {
                    putString("key", "백그라운드 작업 완료")
                }
            }

            // 메인 스레드의 Handler를 통해 메시지 큐에 메시지를 전달합니다.
            mainHandler.sendMessage(msg)
        }.start()
    }
}
```