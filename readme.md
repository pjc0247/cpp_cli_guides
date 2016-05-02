cpp_cli_guides
====

C++ 코드에서 .Net 코드를 호출하는 방법을 알아봅니다.

시작하기
----
C++/CLI를 사용하면 C++에서 관리되는 코드를 실행할 수 있습니다.<br>
이 말은 즉 C++에서 C#등으로 만들어진 API를 호출하는것이 가능하다는 뜻 입니다.<br>
기존 프로젝트에서 옵션을 변경하여 바로 C# 코드 실행이 가능한 상태로 만드는것 또한 가능합니다만, 이부분은 엄청난 제약이 따릅니다.
```
/clr 옵션을 설정할 경우, 대부분의 다른 컴파일 옵션을 사용하지 못합니다.
뭣보다 c++ stdlib중 스레드 관련 부분을 전혀 사용하지 못한다는 점이 가장 큽니다.

예시)
<mutex> is not supported when compiling with /clr or /clr:pure.
```
따라서 아주아주대부분의 경우는 C# API를 C++ 형태로 래핑한 DLL을 만들고, 해당 DLL을 실제 프로젝트에 링크하여 사용하는 방식이 권장됩니다.<br>
<br>
C++/CLI를 사용했을 때 일반적인 .Net 함수 호출은 다음과 같이 수행됩니다.
```cpp
void main() {
  System::Console::WriteLine("Hello World!");
}
```

* 반대로 기존 C++ 코드를 CLR에서 구동 가능한 바이너리로 출력하여 C#에서 C++ 코드를 사용할 수도 있습니다. 하지만 이는 단순히 C#에서 DLL Import로 해결할 수 있으며, 여기서는 이 방법에 대해서는 자세히 다루지 않겠습니다.


관리되는 클래스 만들기
----
클래스 선언 앞에 `ref` 키워드를 붙여 관리되는 클래스로 지정할 수 있습니다.<br>
관리되는 클래스는 다른 관리되는 클래스의 인스턴스를 프로퍼티로 포함할 수 있습니다.
```cpp
ref class ManagedFoo {
public:
	int var = 0;
  
  // C# 과 동일하게 별도 지정하지 않아도 System::Object에서 상속받습니다.
  // 따라서 ToString 을 오버라이드 하는 것 또한 가능합니다.
	System::String ^ToString() override {
		return "MY CUSTOM TOSTRING()";
	}
};
```
관리되는 클래스는 GC에 의해 관리되므로 별도의 `delete` 동작이 필요하지 않습니다.<br>
아래 코드는 관리되는 `ref` 클래스를 생성하는 방법입니다.
```cpp
// new 대신 gcnew 키워드로 생성합니다.
auto foo = gcnew ManagedFoo();

Console::WriteLine(foo);
```

* `!dtor`등의 `ref` 클래스에 대한 자세한 항목들은 이 레포에서 다루고자 하는 주제와 별로 관련이 없으므로 생략합니다.


익셉션
----
C#/.NetAPI 에서는 익셉션으로 상태를 반환하는것이 일반적입니다.<br>
익셉션을 반환할 수 있는 관리되는 메소드를 호출할 때는 아래와 같이 `try~catch` 구문을 사용합니다.
```cpp
try {
  auto dic = gcnew System::Collections::Generic::Dictionary<int, int>();
  
  // 존재하지 않는 키 검색
  int value = dic[1234];
}
catch (Exception ^e) {
	Console::WriteLine(e);
}
```

문자열 변환하기
----
__System::String을 std::string 으로 변환하기__
```cpp
#include <msclr/marshal_cppstd.h>

System::String ^a = "Hello World!";

// cpp string이 반환됩니다.
msclr::interop::marshal_as<std::string>(x);
```

__std::string을 System::String 으로 변환하기__
```cpp
std::string str = "Hello World!";

// managed string이 반환됩니다.
gcnew System(str.c_str());
```

Action 인스턴스 생성하기
----
```cpp
static void TestCallback(int data) {
  printf("Hello : %d\n", data);
}

SomeFunc(gcnew System::Action<int>(&TestCallback));
```

람다 함수 사용하기
----
일반적으로 Action을 생성할 때 함수 포인터는 넘겨줄 수 있지만 람다식은 넘겨줄 수 없습니다.<br>
```cpp
gcnew System::Action<int>([](int data) {
  printf("Hello : %d\n", data);
});
```
하지만 이미 스택오버플로우에 완벽한 해결책이 존재합니다.<br>
[StackOverflow](http://stackoverflow.com/a/26552573)<br>
위 코드가 어떠한 흑마법으로 이루어져 있는지는 [C++ 마스터](https://github.com/jwvg0425) 에게 문의 주세요.
```cpp
SomeFunc(Lambda2Delegate<>() = [](int data) {
  printf("Hello : %d\n", data);
});
```

관리되는 인스턴스를 포함하는 C++ 클래스 만들기
----
원칙적으로는 `ref`가 아닌 일반 클래스는 관리되는 프로퍼티를 포함한 c++ 인터페이스를 만들 수 없습니다.<br>
하지만 아래와 같은 우회로를 이용하여 인터페이스상에는 관뢰되는 프로퍼티를 직접적으로 노출시키지 않지만, 실제로는 가지고 있는 형태로 작성할 수 있습니다.
__MyClass.h__
```cpp
class MyClassData;

class MyClass {
public:
  MyClass();
  virtual ~MyClass();

private:
  MyClassData *data;
}
```
<br>
__MyClass.cpp__
```cpp
#include <msclr/auto_gcroot.h>

class MyClassData {
public:
  msclr::auto_gcroot<System::String^> str;
};

MyClass::MyClass() {
  data = new MyClassData();
  data->str = gcnew System::String("Hello World!");
}
MyClass::~MyClass() {
  delete data;
}
```

관리되는 인스턴스를 주고받는 메소드 작성하기
----
최종적으로 유저에게 노출되는 인터페이스 상에는 관리되는 오브젝트를 주고받도록 작성하면 안되겠지만, 내부적으로 클래스간에는 관리되는 객체를 주고받을일이 분명히 있을 것입니다.<br>
해결책 중 하나는 위처럼 전방선언을 이용한 꼼수를 사용하는것과, 경우에 따라서는 더 간단하게 해결할 수 있는 `reinterpret_cast`를 이용한 방법이 있습니다.<br>
아래에서는 `reinterpret_cast`를 이용한 방법에 대해 설명합니다.
<br>
```cpp
class Doll {
  // TODO : DollFactory가 접근할 수 있도록 friend를 설정합니다. 또는 알아서
private:
  Doll(void *ptr);
}
class DollFactory {
public:
  static Doll *Create() const;
}
```
```cpp
#include <msclr/auto_gcroot.h>

Doll::Doll(void *ptr) {
  ManagedDoll ^doll = *reinterpret_cast<ManagedDoll^*>(ptr);
  /* .... */
}
Doll *DollFactory::Create() const {
  msclr::auto_gcroot<ManagedDoll^> holder(gcnew ManagedDoll());
  return new Doll(&holder.get());
}
```
void *로 관리되는 인스턴스를 넘길 때는 GC에 의해 레퍼런스 되지 않는 점에 주의하세요.


타겟 프레임워크 버전 변경하기
----
이유는 모르겠지만 C#에서와 같이 타겟 .Net 프레임워크 버전을 변경하는 UI가 제공되지 않습니다.<br>
아래 과정을 따라 수동으로 버전을 변경합니다.<br>
* 대상 프로젝트 -> 우클릭 -> 프로젝트 언로드
* 편집 .vcxproj
* `TargetFrameworkVersion` 값을 원하는 값으로 변경
  * 예) v4.6
* 프로젝트 -> 우클릭 프로젝트 다시 로드