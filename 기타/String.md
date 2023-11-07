# String vs StringBuilder vs StringBuffer

<img src="https://github.com/dev-baik/TIL/assets/96613859/4e2521a1-a7bb-41c1-8e95-f47cd11350ff" width="50%" height="50%"/>

- String은 Java에서 가장 널리 사용되는 클래스 중 하나다. 
- StringBuffer 및 StringBuilder 클래스는 문자열을 조작하는 메서드를 제공한다.

## String in Java
1. String 클래스는 문자열을 나타내며 두가지 방법으로 String을 인스턴스화할 수 있다.
```kotlin
String str = "ABC";

String str = new String("ABC");
```
2. 문자열은 Java에서 불변이다. 따라서 멀티스레드 환경에서 사용하기에 적합하고, 데이터 불일치에 대한 우려가 없기 때문에 기능 간 공유가 가능하다.
3. 큰따옴표를 사용하여 문자열을 생성하면 JVM은 먼저 [문자열 풀](https://junhyunny.github.io/java/java-string-pool/)에서 동일한 값을 가진 문자열을 찾는다.  
   - 찾은 경우 풀에서 문자열 객체의 참조를 반환한다. 그렇지 않으면 문자열 풀에 문자열 객체를 만들고 참조를 반환한다.
     - JVM은 서로 다른스레드에서 동일한 문자열을 사용하여 많은 메모리를 절약한다.
4. new 연산자를 사용하여 문자열을 생성하면 힙 메모리에 생성된다. 

    <img src="https://github.com/dev-baik/TIL/assets/96613859/4f43b7bb-cbf0-41e6-8272-9769eeeb8a7f" width="30%" height="30%"/>

5. \+ 연산자는 문자열에 대해 오버로드된다. 이를 사용하여 두 문자열을 연결할 수 있다. 내부적으로는 StringBuilder를 사용하여 이 작업을 수행한다.
6. String은 [equals() 및 hashCode()](https://www.digitalocean.com/community/tutorials/java-equals-hashcode) 메서드를 재정의한다. 두 문자열은 동일한 문자 순서를 갖는 경우에만 동일하다. equals() 메소드는 대소문자를 구분한다. 대소문자를 구분하지 않는 검사를 찾고 있다면 equalsIgnoreCase() 메소드를 사용해야 한다.
7. 문자열은 문자 스트림에 [UTF-16 인코딩](https://lordofkangs.tistory.com/86)을 사용한다.
8. 문자열은 final 클래스이다. `private int hash`를 제어한 모든 필드는 final 필드이다. 이 필드는 hashCode() 함수 값이 포함되어 있다. 해시코드 값은 hashCode() 메서드가 처음 호출된 후 이 필드에 저장되는 경우에만 계산된다. 또한, 일부 계산과 함께 String 클래스의 final 필드를 사용하여 해시가 생성된다.
    - 따라서 hashCode() 메서드가 호출될 때마다 동일한 출력이 발생한다. 호출자 입장에서는 매번 계산이 일어나는 것처럼 보이지만 내부적으로는 해시 필드에 저장되어 있다.

## String vs StringBuffer
- 문자열은 Java에서 불변이기 때문에 연결, 문자열 자르기 등과 같은 문자열 조작을 수행할 때마다 새 문자열을 생성하고 가비지 수집을 위해 이전 문자열을 삭제한다.
- 이는 무거운 작업이며 힙에 많은 쓰레기를 생성한다. 따라서 Java는 문자열 조작에 사용해야 하는 StringBuffer와 StringBuilder 클래스를 제공한다.
- StringBuilder와 StringBuilder는 Java의 변경 가능한 객체이다. 문자열 조작을 위해 append(), insert(), delete(), substring() 메소드를 제공한다.

## StringBuffer vs StringBuilder
- StringBuffer는 Java 1.4까지 문자열 조작을 위한 유일한 선택이었다. 그러나 모든 public 메서드가 동기화된다는 단점이 있다. StringBuffer는 스레드 안전성을 제공하지만 성능 비용이 발생한다.
- 대부분의 시나리오에서는 다중 스레드 환경에서 String을 사용하지 않는다. 따라서 Java 1.5에는 스레드 안전성 및 동기화를 제외하고 StringBuffer와 유사한 새로운 클래스 StringBuilder가 도입되었다.
- StringBuffer에는 substring, length, capacity, trimToSize 등과 같은 몇 가지 추가 메서드가 있다. 그러나 이러한 모든 메서드가 String에도 있으므로 이러한 메서드는 필요하지 않다. 이것이 바로 이러한 메서드가 StringBuilder 클래스에서 구현되지 않은 이유이다.
- StringBuffer는 Java 1.0에서 도입되었지만 StringBuilder 클래스는 StringBuffer의 단점을 고려하여 Java 1.5에서 도입되었다.
- 단일 스레드 환경에 있거나 스레드 안전에 관심이 없다면 StringBuilder를 사용해도 된다. 그렇지 않으면 스레드로부터 안전한 작업을 위해 StringBuilder를 사용하라. 

## StringBuilder vs StringBuffer 성능 비교
- 여러번 add() 메서드를 호출하는 프로그램에서 StringBuffer와 StringBuilder 객체를 사용할 때 동기화로 인한 성능 영향을 확인하기
```java
import java.util.GregorianCalendar;

public class TestString {

	public static void main(String[] args) {
		System.gc();
		long start = new GregorianCalendar().getTimeInMillis();
		long startMemory = Runtime.getRuntime().freeMemory();
        
		StringBuffer sb = new StringBuffer();
		//StringBuilder sb = new StringBuilder();
        
		for(int i = 0; i < 10000000; i++){
			sb.append(":").append(i);
		}
        
		long end = new GregorianCalendar().getTimeInMillis();
		long endMemory = Runtime.getRuntime().freeMemory();
		System.out.println("Time Taken:" + (end - start));
		System.out.println("Memory used:" + (startMemory - endMemory));
	}
}
```

<table>
    <tr>
        <th>i의 값</th>
        <th>StringBuffer (Time, Memory)</th>
        <th>StringBuilder (Time, Memory)</th>
    </tr>
    <tr>
        <td>10,00,000</td>
        <td>808,149356704</td>
        <td>633,149356704</td>
    </tr>
    <tr>
        <td>1,00,00,000</td>
        <td>7448,147783888</td>
        <td>6179,147783888</td>
    </tr>
</table>

- 단일 스레드 환경의 경우에서 StringBuilder가 StringBuffer보다 성능이 더 좋다는 것은 분명하다. 이러한 성능 차이는 StringBuffer 메서드의 동기화로 인해 발생할 수 있다.

> **결론**
> 
> 1. String은 변경할 수 없지만 StringBuffer 및 StringBuilder는 변경 가능한 클래스이다
> 2. StringBuffer는 스레드로부터 안전하고 동기화되지만 StringBuilder는 그렇지 않다. 이것이 StringBuilder가 StringBuffer보다 빠른 이유다.
> 3. 문자열 연결 연산자(+)는 내부적으로 StringBuffer 또는 StringBuilder 클래스를 사용한다.
> 4. 멀티 스레드가 아닌 환경에서 문자열을 조작하려면 StringBuilder를 사용해야 하며 그렇지 않으면 StringBuffer 클래스를 사용해야 한다.

####  참고 사이트
- https://www.digitalocean.com/community/tutorials/string-vs-stringbuffer-vs-stringbuilder
- https://junhyunny.github.io/java/java-string-pool/
- https://lordofkangs.tistory.com/86
- Kotlin : https://kotlinworld.com/36