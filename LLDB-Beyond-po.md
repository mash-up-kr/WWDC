# LLDB: Beyond "po"

![images/LLDB-Beyond-po/Untitled.png](images/LLDB-Beyond-po/Untitled.png)

LLDB는 런타임 환경에서, 앱을 분석하고 디버깅할 수 있도록 돕는 강력한 도구다.

본 발표는 세 가지 파트로 나눠져 있으며 각각

- 변수를 displaying하는 다양한 방법들

- 어떻게 custom data type들을 format하는지

- 자신만의 파이썬3 스크립트를 활용해서 LLDB를 확장하는 방법

순서로 진행된다.

# 변수를 displaying하는 다양한 방법들

![images/LLDB-Beyond-po/Untitled%201.png](images/LLDB-Beyond-po/Untitled%201.png)

![images/LLDB-Beyond-po/Untitled%202.png](images/LLDB-Beyond-po/Untitled%202.png)

- 먼저 위 View 에서는 에서는 정의한 변수와, 해당 변수의 타입에 관한 정보를 알 수 있다.

![images/LLDB-Beyond-po/Untitled%203.png](images/LLDB-Beyond-po/Untitled%203.png)

- 또한 위와같이 Xcode의 오른쪽 아래의 뷰에 커맨드를 입력하므로서 LLDB와 소통할 수 있다.

- 앱을 조사하는 동안 커맨드를 통해 명령어를 입력하면, 소스코드 내, 변수에 대한 정보를 제공받을 수 있다.

- LLDB는 이를 위한 몇가지 방법을 제공하는데, 각각의 방법엔 서로다른 장단점이 존재한다.

## PO (Print Object)

- PO는 Object 변수의 정보를 얻기 위한 명령어다.

![images/LLDB-Beyond-po/Untitled%204.png](images/LLDB-Beyond-po/Untitled%204.png)

- 이름 그대로 "po 변수명"형태로 입력하면, 해당 변수의 Object명과 정보들을 얻을 수 있으며, 위와같은 예가 가장 일반적인 사용법이다.

![images/LLDB-Beyond-po/Untitled%205.png](images/LLDB-Beyond-po/Untitled%205.png)

- 이때 만약, 어떠한 Object에 CustomDebugStringConvertible 프로토콜을 상속시키고, debugDescription: String 프로퍼티가 적절한 값을 반환하도록 하면,

- 위 결과 값처럼 po 명령어를 사용했을 때, Object명이 아닌 원하는 텍스트를 출력하도록 할 수도 있다.

![images/LLDB-Beyond-po/Untitled%206.png](images/LLDB-Beyond-po/Untitled%206.png)

- CustomDebugStringConvertible은 최상위 레벨에 관한 변경만 이루어지며 만약 부수적인 부분을 바꾸고 싶다면, CustomReflectable 프로토콜을 상속시키도록 하자.

![images/LLDB-Beyond-po/Untitled%207.png](images/LLDB-Beyond-po/Untitled%207.png)

- po를 잘 활용하면, 단순 변수를 출력하는것 뿐만아니라, 위처럼 값의 uppercase나, 배열의 정렬된 형태를 출력하는 것도 가능하다.

![images/LLDB-Beyond-po/Untitled%208.png](images/LLDB-Beyond-po/Untitled%208.png)

- command alias를 활용하면 직접, po를 커스텀해서 사용할 수 도 있다.

![images/LLDB-Beyond-po/Untitled%209.png](images/LLDB-Beyond-po/Untitled%209.png)

po의 작동방식에 대해서 알아보자.

LLDB는 먼저 expression을 컴파일 가능한 코드로 변환한다.

그리고 Swift 컴파일러를 통해, 디버그되고있는 프로그램의 해당 context에서 해당 코드를 실행시킨다. (1번째 줄)

그리고 결과값이 나오면, 해당 결과 값에 접근할 수 있는 코드를 마찬가지로 생성하며, 이를 컴파일하고 실행한다. (2번째줄)

String 형태의 결과 값을 얻고 화면에 출력한다.

### p

![images/LLDB-Beyond-po/Untitled%2010.png](images/LLDB-Beyond-po/Untitled%2010.png)

이제 p 관련 예제를 보자.

언뜻보면, 결과창은 미세하게 달라졌을 뿐, po와 비슷한 내용을 출력하고 있다.

대신 p는 오브젝트명을 $R0이라는 형태로 출력하는걸 볼 수 있다.

이는 LLDB의 특별한 Convention이다. 이 이름은 오름차순으로 부여받는다. 예를들면 $R0, $R1, $R2 같이 말이다.

이러한 이름들은 LLDB의 나중에 다른 표현식에 사용된다.

![images/LLDB-Beyond-po/Untitled%2011.png](images/LLDB-Beyond-po/Untitled%2011.png)

$R0같은 이름은 그 자체로도 표현식에 사용될 수 있고 내부 프로퍼티도 출력할 수 있다. 

![images/LLDB-Beyond-po/Untitled%2012.png](images/LLDB-Beyond-po/Untitled%2012.png)

Swift에서는 꼭 Type이 고정될 필요가 없다.

위에선 Trip은 Activity 프로토콜을 상속하고 있고, cruise는 Activity타입의 변수로 선언되어있다.

하지만 p 커맨드를 통해 출력했을 때는, 가장 정확도가 높은 타입인 "Trip"형태로 출력되는 걸 볼 수 있다.

이를 Dynamic Type resolution이라고 한다.

![images/LLDB-Beyond-po/Untitled%2013.png](images/LLDB-Beyond-po/Untitled%2013.png)

일반적인 소스코드에서 위처럼 하위타입에 포함된 프로퍼티를 출력하려고 하면 에러가 발생한다. 

p 커맨드 역시 비슷한 에러가 뜬다.

이유는 아까 설명했듯이, p 표현식 자체가 해당 소스코드가 있는 곳에서 compile되기 때문이다. 그렇기 때문에 해당 소스코드에 있는 값 밖에 볼 수 없다. (static one)

이 땐 아래처럼, 표현식에 타입캐스팅을 넣어주면 된다.

### V

v도 위의 두개와 마찬가지로 "frame variable"이라는 명령어의 alias다.

v는 p나 po와는 다르게 컴파일이나 실행을 전혀 하지 않는다. 그렇기 때문에 속도가 상대적으로 빠르다.

대신, 코드 실행 필요한, 연산프로퍼티 같은건 출력이 불가능하다.

![images/LLDB-Beyond-po/Untitled%2014.png](images/LLDB-Beyond-po/Untitled%2014.png)

v의 동작을 살펴보면, 단순히 메모리에 있는 변수를 읽고 포맷해서 제공할 뿐이며, 만약 변수의 내부 필드를 참조하고 싶다면, 이 과정을 여러번 반복한다.

이는 Dynamic type resolution을 한번만 하는 p와의 가장 큰 차이점이다.

![images/LLDB-Beyond-po/Untitled%2015.png](images/LLDB-Beyond-po/Untitled%2015.png)

이 예제는 p대신 v를 사용해야하는 상황이다.

별도의 타입변환 없이도 위처럼 Dynamaic type resolution을 해준다.

### 정리

![images/LLDB-Beyond-po/Untitled%2016.png](images/LLDB-Beyond-po/Untitled%2016.png)

1. 어떻게 Object가 Present 되는지

    - po는 Object description을 사용한다.

    - p와 v는 data formatter를 사용한다.

2. 결과값이 어떻게 계산되는지

    - po와 p는 표현식을 컴파일해서 full language에 접근한다.

    - v는 자신만의 syntax를 가지고 표현식을 interpret하며, dynamic type resolution에 있어서도 각각의 step을 interpret한다.

## Customizing Data Formatter

![images/LLDB-Beyond-po/Untitled%2017.png](images/LLDB-Beyond-po/Untitled%2017.png)

Data Formatter를 커스터마이징해서, 콘솔창에 깔끔하게 출력되도록 해보자.

커스터마이징엔 위의 세가지 방법이 사용된다.

### Filters

![images/LLDB-Beyond-po/Untitled%2018.png](images/LLDB-Beyond-po/Untitled%2018.png)

Filter를 사용하면, 위처럼 Trip타입의 cruise라는 변수에서, name프로퍼티만 출력하도록 할 수 있다.

### String summaries

![images/LLDB-Beyond-po/Untitled%2019.png](images/LLDB-Beyond-po/Untitled%2019.png)

String summaries를 추가해서 원하는 설명을 원하는 포맷으로 출력할 수도 있다.

![images/LLDB-Beyond-po/Untitled%2020.png](images/LLDB-Beyond-po/Untitled%2020.png)

이 때 달러싸인($)을 사용할 수 있고, 해당 변수를 참조할때는 var를 사용한다.

문제는 Formatter는 연산 프로퍼티 등에 접근할 수 없기 때문에, 배열의 count같은 값에 접근할 수 없다. 그래서 위 예제처럼 destination의 마지막 인덱스(2)를 하드코딩 해줘야 한다는 불편함이있다.

![images/LLDB-Beyond-po/Untitled%2021.png](images/LLDB-Beyond-po/Untitled%2021.png)

![images/LLDB-Beyond-po/Untitled%2022.png](images/LLDB-Beyond-po/Untitled%2022.png)

위와같은 문제를 해결하기 위해, LLDB는 파이썬으로 코드를 작성해서 import하는 방식으로도 커스터마이징 수단을 제공한다.

위 예제는 Trip.py라는 파이썬파일을 작성하여 LLDB내에서 import하는 과정이다.
