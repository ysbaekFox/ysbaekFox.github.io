---
published: true
layout: single
title: "Undefined Behavior - Back To Basics"
category: cppcon
tags:
comments: true
sidebar:
  nav: "mainMenu"
---  
* * *

# Overview

Undefined Behavior는 Compiler에 정의되지 않은 동작으로서 프로그램이 어떻게 동작할지 알 수 없는 것을 의미합니다.

우리는 Undefined Behavior에 대해 충분히 이해하고 있다고 생각합니다.(실상은 그렇지 않지만요.)

또한, Undefined Behavior로 인한 Bug를 경험해 보았을 것이고
알 수 없는 Bug의 원인을 찾기 위해 반복적인 Test와 디버깅, Code Review로 고생해본 경험이 있을 것 입니다.

오늘은 CppCon에서 "Back To Basics (기존으로 돌아가자)"라는 주제로 Undefined Behavior에 대한 강의를 보고 내용을 정리하였습니다.
주제 그대로 기본으로 돌아가서 알고 있던 내용은 복기하고, 모르던 내용은 숙지할 수 있는 시간이 되었으면 합니다.

참고로, CppCon은 1년에 한번씩 열리는 C++ Conference로 모든 강의들이 Youtube에 올라와 있습니다.
- [Undefined Behavior - Back To Basics 유튜브 강의](https://www.youtube.com/watch?v=NpL9YnxnOqM&list=PLHTh1InhhwT4TJaHBVWzvBOYhp27UO7mI&index=4)
- [강의 자료 PDF](https://www.copperspice.com/pdf/Undefined-Behavior-CppCon-2021.pdf)

#### Undefined Behavior에 대한 흔한 오해
* * *

Undefined Behavior에 대한 흔한 오해들이 있습니다.
  
- Code Review를 통해서 Undefined Behavior이 발견할 수 있다.
- Undefined Behavior를 디버깅하는 것은 약간의 연습만으로 충분하다.
- 좋은 테스트(단위 테스트 ~ 통합 테스트)는 Undefined Behavior를 잡아낼 수 있다.
- C++ Standard Committee은 Undefined Behavior를 C++에서 제거하기 위해 노력하고 있다.
- 성능이 좋은 컴파일러는 Undefined Behavior을 오류로 보고한다.
- 경험이 많은 숙련된 개발자는 Undefined Behavior Code를 만들지 않는다.
  
위의 목록은 모두 우리가 잘못알고 있는 것들 입니다. 언뜻 보면 동의할 수 있는 말들도 있어 보이지만 모두 사실이 아닙니다.
  
뒤에서 설명할 내용들을 듣고 나면 그 오해들이 해결될 것 입니다.

#### Compiler vs Application Developer
* * *

우리는 Undefined Behavior를 2가지 관점에서 바라볼 수 있습니다.
하나는 C++ Compiler 개발자의 관점이고, 다른 하나는 이미 개발된 C++ Compiler를 사용하는 Application 개발자의 관점 입니다.

우리는 당연히 후자에 속합니다. 그러므로 Undefined Behavior의 모든 측면을 이해할 필요는 없을 것 입니다.

**Compiler Developer**
- 목표는 모든 기회를 활용하여 최적화하는 것.
- Undefined Behavior은 재미있는 이론적 토론이 될 수 있음.
- Undefined Behavior의 모든 측면을 이해하는 것은 필수적임.
- Undefined Behavior을 간과하는 것은 성능에 영향을 줄 수 있음.

**Application Developer**
- 목표는 Undefined Behavior이 0인 코드를 생성하는 것.
- Undefined Behavior은 벅차고, 두렵게 만드는 토론이 될 수 있음.
- Undefined Behavior을 방지하는 방법에 대한 이해는 필수 사항.
- Undefined Behavior을 무시하는 것은 매우 위험.

<br>

# Why?
* * *

그렇다면 Undefined Behavior라는 것은 정확히 왜 발생하는 것일까요?

C++ 표준 위원회에서는 올바른 C++ 프로그램에는 Undefined Behavior가 없어야 한다고 말합니다.
그래서 컴파일러들은 최적화하는 과정에서 Undefined Behavior가 없다는 것을 가정하고 동작하고 있습니다.
그로인해 컴파일러가 코드를 번역 및 최적화하는 과정에서 예기치 않은 동작들이 발생하는 것입니다.

또한 C++에는 다양한 Behavior들이 있는데 어떤 종류가 있는지 숙지할 필요가 있습니다.
왜냐하면, Undefined Behavior인 것과 아닌 것을 구분할 필요가 있기 때문입니다.

아래의 동작들은 제가 임의로 명명한 것이 아니라 cppreference에서 정확히 구분하고 있습니다. 
(참고로 실제 페이지에서는 syntax error와 linking error를 포함하는 ill-formed 유형이 있지만 해당 내용은 제외시켰습니다.)
- [cppreference - Undefined Behavior](https://en.cppreference.com/w/cpp/language/ub)
<br>
<br>

**Defined Behavior**
- 코드가 분명하고 예상된대로 동작함

  ```c++
  int sum = 17 + 8;
  printf(“Welcome to CppCon 2021”);
  auto [first, second] = getPair();
  ```
<br>

**Implementation Defined Behavior**
- 코드가 여러가지 의미로 해석 될 수 있음. (플랫폼에 따라서)
- 컴파일러는 그 중 하나를 선택하고 그 것을 문서화해야만 함.

  ```c++
  if ( sizeof(int) < sizeof(long) ) 
  {
      std::cout << "sizeof(int) < sizeof(long)" << std::endl;
  }
  ```

  ```c++
  // MSVC (VS2019)
  4
  4

  // GCC 9.3.0 (WSL Ubuntu)
  4
  8
  sizeof(int) < sizeof(long)
  ```
<br>

**Unspecified Behavior**
- 코드가 여러가지 의미로 해석 될 수 있음.
- 컴파일러는 그 중 무작위로 아무 동작이나 해도 됨 (문서화 X)
- Undefined Behavior보다는 안전함.
  ```c++
  // 단 아래 동작은 C++17 이후부터는 동일하게 판단하는 것이 표준
  if ( "str1" == "str2" )
  { 
      std::cout << "str1 == str2" << std::endl;
  }    
  ```
<br>

**Undefined Behavior**
- 그 밖의 C++ 표준에 정의되지 않은 모든 동작.
- 심지어 컴파일러는 정의되지 않은 동작에 대해 진단할 필요조차도 없습니다.
(물론 단순한 상황은 진단되는 경우가 있습니다.)

<br>

# Example Codes
* * *

Undefined Behavior가 포함된 코드를 보면서 어디가 Undefined Behavior인지 찾아보도록 .

#### Example 1
* * *
std::vector<T>의 end를 지나서 값을 read하면 어떤일이 발생할까요?<br>

```c++
std::vector<std::string> name = { "tiger", "horse", "ostrich", "gerenuk", "jodankee" };
std::cout << *name.end() << std::endl;
```

C++ 표준에서는 STL Container의 끝을 넘어 접근하는 것을 Undefined Behavior으로 분류하고 있습니다.

#### Example 2
* * *
아래 코드도 한번 확인해보겠습니다.
어떤 라인에서 Undefined Behavior가 발생하는지 찾을 수 있으실까요?

```c++
int* varA = nullptr;
*varA = 17;

int varB;
varA = &varB;

std::cout << *varA << std::endl;
std::cout << varB << std::endl;
```
<details>
<summary> 정답 </summary>
<div markdown="1">

```c++
int* varA = nullptr;
*varA = 17; // dereferencing a null pointer is UB

int varB;
varA = &varB; // address of varB is valid

std::cout << *varA << std::endl; // dereference is valid, 
                                 // but accessing an uninitialized variable is UB
                                 
std::cout << varB << std::endl;  // accessing an uninitialized variable is UB
```

</div>
</details>

<br>
여러군데에서 Undefined Behavior가 발생하고 있습니다. 발생하는 Undefined Behavior의 종류는 2가지로 null pointer 역참조 그리고 초기화되지 않은 변수 사용입니다.

#### Example 3
* * *
bit 연산을 하는 간단한 코드 입니다.  
아래 코드에서 Undefined Behavior를 찾을 수 있을까요? 

```c++
int x = 1;
int y = x << 34;
std::cout << y << std::endl;
```
  
line 2은 Undefined Behavior를 유발합니다.  
왜냐하면 left shift 연산의 결과는 대상 유형의 범위를 초과하기 때문입니다.

그래서 Left Shift 연산에서 Undefined Behavior를 방지하려면 사용자는 항상 left shift의 양이 
대상 유형의 bit수보다 적고 shift 결과가 대상 유형으로 표시될 수 있는지 확인해야 합니다.


#### How is Undefined Behavior Defined in C++
* * *

더 많은 예제를 살펴보기 전에 잠깐 돌아와서 C++에서 
Undefined Behavior를 어떻게 정의하고 있는지 살펴보겠습니다.

- Undefined Behavior는 소스 코드를 실행한 결과에 대한 동작이 C++ 표준에 정의되어 있지 않은 것이다.
- Undefined Behavior를 유발하지 않는 코드를 작성하는 것은 온전히 프로그래머의 책임이다.
- Undefined Behavior로부터 자유로운 코드로 작성된 프로그램만이 올바르게 동작한다.
- C++ Standard는 Undefined Behavior가 없는 경우에만 올바른 동작을 보장한다.

즉 **Undefined Behavior는 C++ 표준 및 컴파일러에서 어떤 보장도 해주지 않고 
온전히 프로그래머가 짊어져야할 책임**인 것 입니다. 그렇기 때문에 우리는 Undefined Behavior에 대해 반드시 숙지해야만 합니다.

#### 일반적인 Undefined Behavior 목록
* * *

C++ 표준 위원회는 매우 특수한 경우를 제외하고는 Undefined Behavior을 계속해서 추가하고 있기 때문에 모든 Undefined Behavior 리스트를 숙지하는 것은 사실상 불가능 합니다.
  
하지만 우리는 일반적인 Undefined Behavior 사례에 대해서는 정리할 수 있을 것 입니다.
아마도 코드상에서 발생하는 Undefined Behavior의 90% 이상은 일반적인 경우일 것이라고 생각합니다.

일반적인 Undefined Behavior로 분류 될 수 있는 경우들을 정리해보았습니다.
아마도 아시는 내용도 있고, 몰랐지만 관습적으로 기피했던 내용도 있고 아예 모르고 있던 부분도 있을 것이라고 생각합니다.

- STL Container의 end 부분을 넘어서 접근
- nullptr 역참조
- 초기화 되지 않은 변수 사용
- 생성자/소멸자에 순수 가상 함수를 호출
- 객체가 소멸된 뒤에 사용 (메모리 해제 후 사용)
- 호환되지 않는 유형에 대한 포인터를 캐스팅한 다음 캐스팅한 결과값을 사용
- side effect가 없는 무한 loop
- 문자열 상수 혹은 다른 상수 객체를 수정하는 것
- 반환형이 있는 함수에서 반환 실패
- Thread Unsafety
- 0으로 나눔 연산.
- signed integer overflow (signed long, signed short .. 등)

  <details>
  <summary>Example Codes</summary>
  <div markdown="1">

  ```c++
  // 생성자/소멸자에 순수 가상 함수를 호출

  class Base {
  public:
      Base() {
          pureVirtualFunction();
      }
      virtual void pureVirtualFunction() = 0;
  };

  class Derived : public Base {
      void pureVirtualFunction() {
          std::cout << "pureVirtualFunction called" << std::endl;
      }
  };

  int main() {
      Derived d;
      return 0;
  }
  ```

  ```c++
  // 호환되지 않는 유형에 대한 포인터를 캐스팅한 다음 캐스팅한 결과값을 사용
  int i = 10;
  int* pi = &i;
  char* pc = (char*)pi;
  *pc = 'A';
  ```

  ```c++
  // side effect가 없는 무한 loop

  /* 여기서 side effect란 ?
  - Modifying the value of a variable
  - Accessing or modifying a memory location
  - Performing I/O operations (e.g. reading from or writing to a file or the console)
  - Throwing or catching an exception
  - Calling a function that modifies the state of the program
  - Changing the control flow of the program
  */

  // #1 side effect가 없음.
  // 그러므로 Undefined Behavior
  int i = 0;
  while(true) {
      i++;
  }

  // #2 side effect가 있으므로 Undefined Behavior가 아닐 것 같지만
  // signed int overflow로 인한 Undefined Behavior에 해당함.
  int i = 0;
  while(i != -1){
      i++;
  }
  ```

  ```c++
  // 문자열 상수 혹은 다른 상수 객체를 수정하는 것
  const int x = 5;
  x = 10; // undefined behavior

  const char* str = "Hello";
  str[0] = 'J'; // undefined behavior
  ```

  ```c++
  // signed integer overflow
  // 이 연산의 예상 결과는 부호 있는 int의 최소 표현 가능 값인 -2147483648이지만, 
  // 이 동작은 C++에서 정의되지 않았기 때문에 프로그램은 컴파일러, 플랫폼 
  // 또는 최적화 수준에 따라 다른 결과를 생성할 수 있음.

  int x = 2147483647; // maximum value of a signed int
  x = x + 1; 
  std::cout << x << std::endl;
  return 0;
  ```

  </div>
  </details>


#### Example 4
* * *

이어서 준비한 나머지 예제들을 확인해보도록 하겠습니다.

**부호 있는 정수 산술**
- 결과가 표현 가능한 값의 범위를 벗어나는 경우, "signed integer overflow"가 발생하고 Undefined Behavior 입니다.

부호 없는 정수 산술
- 표준에 따르면 이 작업은 오버플로우가 되지만 정의된 동작 입니다. (예를 들어 255 -> 0이 된다고 표준에 정의되어 있음)

```c++
int volume( int length )
{
    return length * length * length;
}
```

위와 같은 연산 코드는 입력되는 Data 값에 따라 Undefined Behavior가 될 가능성이 있습니다. 
그러므로 일어날 가능성이 낮다고 생각하는 경우에도 데이터 세트 및 입력을 검증하는 것을 권장 합니다.

#### Example 5
* * *

고해성사를 하자면, 저는 실제로 Example 5와 같은 Undefined Behavior를 발생시킨적이 있습니다.
저는 단순히 Crash가 발생하여서 문제를 쉽게 찾을 수 있었지만 단순히 Crash 발생으로 끝나지 않는 경우도 있는 것 같습니다.

그것은 **반환형이 있는 함수에서 반환문을 누락**하는 경우 입니다.
- 이것은 명백하게 Undefined Behavior
- 일부 컴파일러는 경고를 제공하기도 합니다.
- 실행 시간에 일부 Sanitizer 의해 감지되기도 합니다.  
<br>

참고로 프로그램 실행 중에 Undefined Behavior가 발생하게 되면 다음의 Undefined Behavior가 수행될 수 있습니다.
- 매번 true를 반환할 수 있습니다.
- 실행 파일에서 "다음 함수"로 진행할 수 있습니다.
- Crash가 발생할 수 있습니다. (이게 그나마 양반이네요.)

```c++
bool monthOfCppCon21() {
 someData == “October”; 
}
```
  
함수가 매번 true를 반환하거나 엉뚱한 함수가 호출되는 경우에 반환문 누락으로 인해 발생할 수 있다는 것을 숙지하고 있으면 도움이 될 것 같네요.

#### Example 6
* * *

operator[]는 문자열의 index에 대한 참조를 반환합니다.  
이 코드에는 index + 1 및 index + 2가 범위 내에 있는지 확인하는 테스트가 없습니다.  
루프가 문자열 끝에 도달하면 어떻게 될까요?

```c++
std::string inputStr = "class std::vector<int>";
std::string result;
for (int index = 0; index < inputStr.size(); ++index) {
    if (inputStr[index+1] == ':' && inputStr[index+2] == ':') {
        index += 2;
        result = inputStr.mid(index); // expected “vector<int>”
    }
}
```

컨테이너의 범위를 초과해서 접근하지 말하야 하는 것은 다들 알고 있을 것 입니다.  
반복문을 통해 container 내부의 값을 참조할 때는 위와 같은 상황을 항상 고려해야 할 것 입니다.

#### Example 7
* * *

어떤 동작은 컨테이너의 반복자를 무효화시켜버립니다.  
모든 상황에 적용할 수 있는 규칙은 없고, 모든 작업에 대해 타당한지를 확인하는 수 밖에 없습니다. 
그 중 하나를 예시로 들어보겠습니다.

```c++
std::vector<int> myContainer = { 42, 14, 5, 31, 9 };
for (auto &item : myContainer) {
    if (item == 5) {
        myContainer.insert(myContainer.begin(), -5); // line A
    }
}
```

위의 코드에는 범위 기반 루프를 사용하므로 반복자(iterator)가 없습니다.
그럼에도 불구하고 반복자는 insert 동작으로 인해 무의미해집니다. 명백히 Undefined Behavior를 유발하는 동작입니다.
  
과연 이러한 동작이 Crash만으로 끝날까요? 아무도 예상할 수 없습니다.

#### Example 8
* * *
 
아래 예제에서 Undefined Behavior는 어디에서 발생할까요?  
  
라인 B라고 대답하실 수도 있는데. const_cast는 그 자체로는 Undefined Behavior가 아닙니다. 정답은 라인 C 입니다. 
본래 const가 붙어 있던 데이터를 수정하는 것은 Undefined Behavior 입니다.

```c++
const std::string value = “tiger”; // line A
doThing8(value);
void doThing8(const std::string & input) {
    std::string &tmp = const_cast<std::string &>(input); // line B
    tmp = “bear”; // line C, this undefined behavior
}
```

위의 예는 매우 흥미로운 예입니다. 이것은 정말로 분석이 어려운 Undefined Behavior의 종류 중 하나이기 때문입니다.
  
왜 분석이 어려운 예시일까요? 우리가 보고 있는 이 코드는 하나의 코드블럭에 존재하지만, 실제로 라인 A, B, C는 전체 코드 어디든지 분산되어 있을 수 있기 때문입니다.
  
그래서 이것은 매우 진단하기 어려운 Undefined Behavior 종류이고, 이 3개의 라인은 분산되어 있기 때문에 3개를 하나의 조합으로 보고 어느 코드에서 잘못된 것인지 찾아내는 것이 매우 어렵습니다.  
<br>
# 사례 연구(Case Study)
* * *
실제 있었던 사례에 대해 알아보고, 어떤식으로 대응하는 것이 좋은 방법인지를 알아보도록 하겠습니다. 
문제 상황이 주어지고, 해결책과 그 이유에 대해 설명합니다. 다만 각자의 상황이 다르기 때문에 100% 정답이라고 단언할 수는 없겠습니다.
  
우리는 Undefined Behavior를 테스트하는 것에 대해 이야기해보도록 하겠습니다.
  
"나는 나의 코드에 Undefined Behavior가 어디에 있는지 알고 있어"라는 개발자가 있었습니다.
그는 말했습니다. "나의 모든 코드 Undefined Behavior가 있지만 모든 유닛테스트에 문제 없이 동작하고 나의 통제하에 있다."
  
그런데 왠 걸? 코드에서 Undefined Behavior를 제거하는 순간 오히려 유닛테스트가 실패하는 것 입니다.
  
이러한 경우 어떻게 해야할까요?

**Description**
- 개발자가 코드에서 정의되지 않은 동작을 발견했습니다.
- 그러나 모든 단위 테스트를 통과했습니다.
- 응용 프로그램에서 정의되지 않은 동작을 제거했습니다.
- 이제는 오히려 일부 단위 테스트가 실패함을 확인했습니다.
  
우리는 해결을 위한 여러가지 전략을 제시합니다.  

하지만 먼저 명심해야할 것은 우리의 코드가 Undefined Behavior를 가지고 있다는 것은 모든 유닛 테스트가 무의미하다는 것 입니다. (유닛 테스트가 성공하든 실패하든 말입니다.)

**Solution**
- Undefined Behavior를 코드에 다시 넣고 모든 유닛 테스트는 통과시키기
- flaky (약속을 잘 지키지 않아 신뢰하기 어려운 라는 Slang 표현)라고 표시해두고 일단 덮어두기
  
위의 2개는 끔찍한 아이디어들입니다.
반면에 근본적인 원인을 찾기 위해 시도하는 아래의 방안들은 좋은 아이디어들입니다.
  
- 다른 컴파일러나 플랫폼으로 시도해보기
- Sanitizer를 사용하여 테스트 해보기
- 유닛 테스트가 성공할 때까지 디버깅해보기
- 유닛 테스트가 실패하는 원인에 대해 알아내기

 왜냐하면 어디서 발생하는지 왜 발생하는지도 모를 크리티컬한 버그들이 유닛 테스트를 실패하게 만든 원인들 속에서 숨어있기 때문에 
 우리는 그 원인에 대해 반드시 알아내서 잡아낼 필요가 있고 충분히 가치있는 투자입니다.
  
<br>

# Undefined Behavior 해결을 위한 다른 방안들
* * *

그렇다면 일반적인 Undefined Behavior를 숙지하고 또 특별한 Case의 Undefined Behavior까지도 모두 숙지를 해야만 Undefined Behavior를 방지할 수 있게 되는 것일까요?

흔히 발생할 수 있는 Undefined Behavior의 예시에 대해서는 충분히 알아본 것 같으니 이번에는 Undefined Behavior를 예방하기 위한 다른 방안들에 대해 알아보도록 하겠습니다.

#### Tool을 사용하는 방법
* * *
- Address Sanitizer
- Memory Sanitizer
- Undefined Behavior Sanitizer
- Thread Sanitizer

####  Code Reviews
* * *
- 정책적으로 Undefined Behavior를 확인하는 절차를 만드는 것
- 컴파일러의 Warnning Message에 주의를 기울이는 것
- 코드를 여러가지 컴파일러를 사용해 빌드해보는 것
- Undefined Behavior를 Critical한 Bug로 간주하는 마음가짐

<br>

# 마무리
* * *
- Undefined Behavior는 오류로 취급할 수 없습니다.
- Undefined Behavior을 제거하는 것은 가끔 하는 일이 아니고 항상 해야만 하는 일 입니다.
- Undefined Behavior는 무시할 수 없습니다. 
- Undefined Behavior으로 인한 문제는 온전히 개발자의 책임이며, C++이라는 언어를 선탁할 때 이미 그 책임에 동의한 것 입니다 (STL 표준에 의해).
- 그러므로 우리는 Undefined Behavior에 대해 잘 숙지하고 코드에서 배제할 수 있도록 노력해야만 합니다.

<br>

# Q & A
* * *

감사합니다.
