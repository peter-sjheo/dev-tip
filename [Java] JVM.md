# JVM(Java Virtual Machine)

![jvm1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbxKh6U%2FbtqCPzYJhpS%2FoKDKiaPoWqwqU86rf7IVVk%2Fimg.png)

- 대표적인 컴퓨터 아키텍쳐인 인텔 x86아키텍처나 ARM아키텍처와 같은 하드웨어 레지스터 기반으로 동작하는게 아닌, 스택 기반으로 동작하는 가상 머신.
- 운영체제 / 플랫폼에 독립적이다. C/C++은 플랫폼에 종속되어 있기에 플랫폼에 따라 int 자료형의 크기가 변할 수 있다. 이에 비해 JVM은 고정이다.
- GC (Gabage Collection)을 통해 동적으로 할당된 메모리를 자동으로 정리해준다.
- JVM의 역할은 자바 애플리케이션을 클래스 로더를 통해 읽어들여 자바 API와 함께 실행하는 것이다.
- Java와 OS 사이에서 중개자 역할을 수행하여 Java가 OS에 구애받지 않고 재사용 가능하게 해준다.
- 메모리 관리, Garbage Collection(GC를 통해 자원을 관리)을 수행한다. (c++ 등에선 제공하지 않음..)
- 자바 바이트 코드를 실행할 수 있는 주체.


# 개발자가 jvm을 알아야 하는 이유는?

- 한정된 메모리 공간을 효율적으로 사용하여, 고성능의 코드를 생산해내기 위해서임.


# Java 소스코드 대락적인 실행과정.

1. 개발자가 .JAVA 파일을 작성 완료한다.
2. javac (자바컴파일러)를 통해 .java -> .class의 바이트코드로 변환이 일어난다.
3. Class Loader를 통해 byte code를 JVM으로 불러들여 Runtime Data Area에 배치한다.
4. Execution Engine을 통해 불러들인 byte code를 기계어로 번역하여 명령어단위로 실행시킨다. 실행방식에는 인터프리터 방식, JIT-Compile 방식이 있다.
5. 실행과정에서 Runtime Data Area 중 Heap에 할당된 메모리를 Gabage Collector 가 주기적으로 정리한다.

<br>
간단하게(?) 말하면 클래스 로더(Class Loader)가 컴파일된 자바 바이트코드를 런타임 데이터 영역(Runtime Data Areas)에 로드하고, 
실행 엔진(Execution Engine)이 자바 바이트코드를 인터프리터/JIT-Compile 방식등을 통해 기계어로 번역해 실행한다.
그리고 실행과정에서 Gabage Collector가 GC를 주기적으로 수행한다.
여기서 나온 주요 키워드들은 #Class Loader, #Runtime Data Area, #Execution Engine, #Gabage Collector등이 있는데 하나씩 알아보자.
</br>



# Class Loader(클래스로더)

클래스로더는 컴파일 타임이 아닌, 런타임에 클래스를 첫참조하게 될때, JVM내 Runtime Data Area에 .class파일을 로딩 시키고, 링크 과정을 통해 관련 클래스들을 연결하는 과정을 거친다.
여기서 키워드는 다음 2가지라고 볼수 있겠다
- 컴파일 타임이 아닌 런타임 중 클래스 첫 참조 시 .class 파일을 JVM에 로딩.
- JVM 내 Runtime Data Area에 바이트코드를 배치.
조금더 상세하게 로딩 과정을 살펴보자.



![loader](https://d2.naver.com/content/images/2015/06/helloworld-1230-3.png)

클래스 로더가 아직 로드되지 않은 클래스를 찾으면, 위의 그림과 같은 과정을 거쳐 클래스를 로드하고 링크하고 초기화한다.
- Loading : 클래스를 파일에서 가져와 Runtime Data Area에 로드한다.
- Verifying : 로드한 클래스가 JLS (자바 언어 명세(Java Language Specification)) 에 부합하는지 검증하는 단계를 거친다. 
- Preparing : 클래스가 요구하는 메모리 공간을 각자의 메모리 영역에 할당한다.
- Resolving : 클래스의 심볼릭 레퍼런스를 다이렌트 레퍼런스로 변경한다.
- Initializing : 클래스 변수들을 적절한 값으로 초기화 한다.


# Runtime Data Area (JVM이 운영체제에 할당받는 메모리 영역)
런타임 데이터 영역은 6개의 영역으로 나눌 수 있다.
<br>
#Heap, #Method Area, #Runtime Constant Pool, #JVM Stack, #Native Method Stack, #PC Register 인데,
Heap, Method Area, Runtime Constant Pool은 모든 스레드가 공유해서 사용하며,
Stack, Native Method Area, PC Register는 스레드별로 할당된다.
각자에 대해서 알아보자.
</br>
### PC Register
- 스레드마다 1개씩 있으며 스레드 시작 시 생성된다. 컴퓨터구조의 레지스터와 유사하게, 현재 실행중인 JVM명령의 주소를 가진다. 

### JVM Stack
- 스레드마다 1개씩 있으며, 스레드 시작 시 생성된다. Stack Frame을 push/pop 하는 역할만을 담당하는데, 자바 소스코드 기준으로 

```Java
e.printStackTrace();

```
로 오류발생한 메소드 CallStack을 찍을때, 각각의 라인은 하나의 스택 프레임을 표현한다.
하나의 스택 프레임에 저장되는 정보는 해당 Method의 매개변수, 지역변수, 임시변수 그리고 어드레스(메소드 호출 한 주소)등이 있는데, Method 종료시 메모리 공간이 사라진다.

### Native Method Stack
- 자바 외의 언어로 작성된 네이티브 코드를 위한 스택으로, JNI를 통해 호출하는 C/C++등의 코드 수행을 위한 스택이다.

### Method Area 
- 메소드 영역은 모든 스레드가 공유하는 영역으로 JVM이 시작될때 생성된다. JVM이 읽어들인 클래스, 인터페이스에 대한 런타임 상수 풀, 필드/메서드 정보, Static 변수, 메서드의 바이트코드등을 보관한다.
오라클 핫스팟 JVM에서는 Permanent Area, Permanent Generation(PermGen)이라고도 한다.

<br>

실제 개발 시, Permanent Space와 관련된 오류를 접할 수도 있었는데, 아래와 같은 오류이다.

</br>

```Java
java.lang.OutOfMemoryError : PermGen space 
```

Method Area(permanent space) 영역은 GC의 대상에서 제외 되기 때문에, 클래스들이 로딩될때 더이상 Permanent Space에 새로운 메모리를 할당 할 수 없을 경우 오류가 발생한다.
> Java 8에서 Metaspace가 도입되면서 Static Object 및 상수화된 String Object를 heap 영역으로 옮김으로써, 최대한 GC가 될 수 있도록 하였다.

### Runtime Constant Pool
- 메서드 영역에 포함되는 영역이지만, JVM 동작에서 가장 핵심적인 역할을 수행하기 때문에 따로 알아 두면 좋을거 같다.
- 각 클래스와 인터페이스의 상수들뿐 아니라 메서드, 필드에 대한 모든 레퍼런스를 담고 있는 테이블이다.
- 어떤 메소드/필드를 참조하게 될때 JVM은 런타임 상수풀을 통해서 메소드/필드의 메모리 주소를 찾아 참조하게 된다.

### Heap
- 인스턴스/객체를 저장하는 공간으로 GC의 대상이 되는 영역이다. JVM성능 이슈에서 가장많이 언급되는 공간이다.
- new 연산자등으로 생성된 객체 또는 배열등이 저장되는 영역이다. (C/C++과 동일)
- Heap영역은 아래 그림과 같이 크게 봤을때는 Young Generation 영역, Old Generation 영역 2개
상세히 봤을때는 Eden, Survivor1/2(Young Generation), Old, Permenent 영역으로 각각 나뉘어진다.

![loader](https://mirinae312.github.io/img/jvm_memory/JVMHeap.png)

#### Young Generation Area 
- 모든 객체는 Young Generation에 생성되며, Young Generation이 Old Generation보다 크기가 상대적으로 적기때문에
GC를 수행하는데 좀 더 적은 시간이 걸린다. 이때문에 Young Generation에서 수행되는 GC를 Minor GC라고도 한다.
- Young Generation은 Eden, Survivor1/2 3개의 영역으로 구성된다. 모든 객체는 Eden영역에서 처음 생성되며 점차 Survivor영역으로 이동한다.

##### Eden 영역(Young 영역)
- 새로 생성된 모든 객체가 처음에 위치하는 영역을 의미한다. 객체가 생성되면 무조건 Eden Space에 생성된다. 
- Eden영역은 모든 객체가 거쳐가는 곳이기 때문에 상대적으로 Survivor 영역들보다 크게 설정된다. 
- Eden 영역에서 GC가 이루어지고 난뒤에 살아남은 객체들은 Survivor 1 영역으로 이동한다.

##### Survivor 영역(Young 영역)
- Survivor 1 영역에서 Minor GC가 수행된 뒤, 살아남은 객체들은 Survivor 2 영역으로 이동한다.
- 만약 Survivor 2 영역에서의 Minor GC 이후에도 객체가 살아남는다면 Old Generation 영역으로 넘어가게 된다.
- (Survivor1/2간의 이동은 한번만 발생하는것은 아니고, 여러번 수행된다. 여러번 수행 이후에도 남아 있을 경우 Old Generation으로 넘어건다.)

#### Old Generation Area
- Old Generation은 Young Generation보다 상대적으로 매우 큰 공간으로, GC를 수행하는데 많은 시간이 걸린다. 따라서 Old Generation에서의 GC는 상대적으로 적게 수행된다.
- 따라서 Old Generation에서 수행되는 GC를 Major GC라고 통칭한다.
- Major GC가 일어날 경우, GC를 진행하는 Thread외에 모든 Thread는 멈추게 된다. 이를 stop-the-world라고 한다. (당연히 이로인해 서버가 죽거나 할 수 있음..)
- GC를 튜닝하는 이유는 Stop-the-world의 시간을 최소한으로 줄이기 위함이다.


### Gabage Collection
- 위에서 설명한 바와 같이 자바의 메모리 관리법중 하나로 Heap영역에 동작으로 할당했던 메모리를 주기적으로 제거해주는 과정을 뜻한다.
- C/C++ 에서는 GC가 없어서 수동으로 직접 메모리 할당(malloc), 해제(free)를 선언해줘야 했다.
- 가비지 컬렉션 대상은, Heap영역에 할당된 객체의 레퍼런스가 존재할 경우 Reachable, 더 이상 없을경우 Unreachable 로 구분하고, Unreachable인 대상들을 수거해간다.

<br>
객체의 레퍼런스가 존재하는 경우라 함은, JVM Stack이나 Method Area에서 생성된 지역변수, Static 변수등이 어떻게 Heap에 생성된 객체를 참조하는지 알아야한다.
Stack이나 Method Area에서는 Heap영역에 생성된 객체의 가상 메모리 주소를 참조하는 형식으로 구성된다.
</br>

이런 환경에서, 예를들어, 메소드가 끝나는 등의 원인으로 인해 Stack에서 StackFrame을 pop 하거나 하는 일이 발생하게 될 경우,
Stack 영역에서 Heap 영역의 메모리 주소를 참조 있는 변수가 Stack에서 삭제된다.
이렇게 될 경우 reference count가 0이 되어, Heap영역에 할당된 객체는 Unreachable 상태가 된다.
이러한 객체들을 주기적으로 가비지 컬렉터가 제거해준다.


Serial GC
서버의 CPU 코어가 1개일 때 사용하기 위해 개발된 가장 단순한 GC
GC를 처리하는 쓰레드가 1개 (싱글 쓰레드) 이어서 가장 stop-the-world 시간이 길다
Minor GC 에는 Mark-Sweep을 사용하고, Major GC에는 Mark-Sweep-Compact를 사용한다.
보통 실무에서 사용하는 경우는 없다 (디바이스 성능이 안좋아서 CPU 코어가 1개인 경우에만 사용) 
GC Algorithm의 종류에는 여러가지가 있는데, 이는 별도로 작성할 예정이다.


