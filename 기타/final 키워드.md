# final 키워
## final 필드
- final을 가장 많이 사용하는 곳은 필드(전역 변수)이다.
- final을 필드에 사용하면 그 필드는 더 이상 수정이 불가능하다.
- final 필드에 초기값을 설정할 수 있는 방법은 단 2가지이다.
  1. 필드 선언 시 초기값 설정
  2. 생성자에게 초기값 설정
- 초기화되지 않은 final 필드가 남아있다면 컴파일 에러가 발생한다.
```java
class FinalField {
    final String str = "수정불가";
    
    public void method() {
        str = "수정"; // 컴파일 에러 : 수정 불가
    }
}
```
- final은 수정할 수 없기 때문에 인스턴스 전역에서 값을 수정할 필요가 없는 동일한 데이터를 가져야 할 때 사용된다.
- final 필드는 static과 자주 쓰인다.
  - WHY) 인스턴스를 생성하지 않고 고정된 값(상수)을 쉽게 호출하여 사용하기 위해서이다.

## final 클래스
- final class는 더 이상 상속이 불가능하다는 것을 의미한다.
- 주로 보안상의 이유로 사용되는데 중요한 class의 자식 class를 생성하여 해당 class를 통해 시스템을 파괴할 수 있기 때문에 이를 방지하기 위해서 사용된다.
  - class 상속이 불가능하므로 내부의 모든 메서드들도 오버라이딩이 불가능하다.
```java
final class FinalClass()
class FinalTest extends FinalClass {} // 컴파일 에러
```

## final 메서드
- final method는 오버라이딩을 금지할 때 사용된다.
- 주로 부모 클래스에 정의한 메서드 기능을 자식 클래스가 그대로 쓰도록 하기 위해 사용된다.
```java
class FinalMethod {
    final void sum() {}
}

class FinalTest extends FinalMethod {
    void sum() {} // 컴파일 에러
}
```

#### 참고 사이트
- https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=heartflow89&logNo=220964832916
- https://velog.io/@paki1019/final-%ED%81%B4%EB%9E%98%EC%8A%A4-final-%EB%A9%94%EC%86%8C%EB%93%9C-final-%ED%95%84%EB%93%9C%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0