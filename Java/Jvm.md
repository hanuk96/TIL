# JVM
JVM (Java Virtual Machine)
- 자바 가상 머신으로 자바 바이트 코드(.class 파일)를 OS에 특화된 코드로변환(인터프리터와 JIT 컴파일러)하여 실행한다.
- 바이트 코드를 실행하는 표준(JVM 자체는 표준)이자 구현체(특정 밴더가 구현한 JVM)로 특정 플랫폼에 종속적임
- JVM 스팩: https://docs.oracle.com/javase/specs/jvms/se11/html/
- JVM 밴더: 오라클, 아마존, Azul, ...

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/bce3cf13-34bc-41cc-8a24-65fa84eb3cb3">

클래스 로더 시스템
- .class 에서 바이트코드를 읽고 메모리에 저장한다.
- 로딩: 클래스 읽어오는 과정
- 링크: 레퍼런스를 연결하는 과정(실제 Heap에 있는 인스턴스를 가리키도록)
- 초기화: static 값들 초기화 및 변수에 할당

메모리
- Heap 영역에는 객체를 저장하며, 공유 자원이다.
- Stack영역에는 쓰레드마다 런타임 Stack을 만들고,그 안에 메소드호출을 Stack 프레임이라 부르는 블럭으로 쌓는다. 쓰레드를 종료하면 런타임 Stack도 사라진다.
- PC(Program Counter) 레지스터: 쓰레드 마다 쓰레드 내 현재 실행할 Stack 프레임을 가리키는 포인터가 생성된다.
- 네이티브 메소드 Stack은 기계어로 작성된 프로그램을 실행시키는 영역이다. 또한 자바 이외의 언어(C, C++, 어셈블리 등)로 작성된 네이티브 코드를 실행하기 위한 공간임
- 메모리 영역에는 클래스 수준의 정보 (클래스 이름, 부모 클래스 이름, 메소드, 변수, static) 저장 공유 자원이다.

실행 엔진
- 인터프리터: 바이크 코드를 한줄씩 실행한다.
- JIT Compiler: 인터프리터 효율을 높이기 위해, 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 모두 네이티브 코드로 바꿔둔다. 그 다음부터 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용한다.
- GC(Garbage Collector): 더이상 참조되지 않는 객체를 모아서 정리한다.

JNI(Java Native Interface)
- 자바 애플리케이션에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법 제공
- Native 키워드를 사용한 메소드 호출
  
네이티브 메소드 라이브러리
- C, C++로 작성 된 라이브러리

### 클래스 로더
로딩, 링크, 초기화 순으로 진행된다.

<img width="800" alt="image" src="https://github.com/hanuk96/TIL/assets/12428689/822b4c22-a2bb-421c-8cbc-90623d3139d7">

로딩
- 클래스 로더가 .class 파일을 읽고 그 내용에 따라 적절한 바이너리 데이터를 만들고 “메소드” 영역에 저장한다.
- class, interface, enum, method, variable
- FQCN: 해당하는 인스턴스의 계층를 모두 포함한 것
```java
일반적인 사용법
String s = new String();
// FQCN
java.lang.String s = new java.lang.String();
```
- 로딩할 대상을 찾는 순서로 부트스트랩 -> 플랫폼 -> 어플리케이션 클래스 로더를 탐색한다.
- 부트 스트랩 클래스 로더
  - JAVA_HOME\lib에 있는 코어 자바 API를 제공한다. 최상위 우선순위를 가진 클래스 로더
- 플랫폼 클래스로더 클래스 로더
  - JAVA_HOME\lib\ext 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다.
- 애플리케이션 클래스 로더
  - 애플리케이션 클래스패스(애플리케이션 실행할 때 주는 -classpath 옵션 또는 java.class.path 환경 변수의 값에 해당하는 위치)에서 클래스를 읽는다.
  
링크
- 로딩이 끝나면 해당 클래스 타입의 Class 객체를 생성하여 “Heap" 영역에 저장.
- Verify, Prepare, Reolve(optional) 세 단계로 나눠져 있다.
  - Verify: 바이트코드의 수정이 있는지, .class 파일 형식이 유효한지 체크한다.
  - Preparation: 클래스 변수(static 변수)와 기본값에 필요한 메모리를 준비한다.
  - Resolve: 심볼릭 메모리 레퍼런스를 메소드 영역에 있는 실제 레퍼런스로 교체한다.
```java
Foo foo = new Foo();
```

초기화
- Static 변수의 값을 할당한다.
