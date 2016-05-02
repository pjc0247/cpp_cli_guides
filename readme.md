cpp_cli_guides
====

C++ �ڵ忡�� .Net �ڵ带 ȣ���ϴ� ����� �˾ƺ��ϴ�.

�����ϱ�
----
C++/CLI�� ����ϸ� C++���� �����Ǵ� �ڵ带 ������ �� �ֽ��ϴ�.<br>
�� ���� �� C++���� C#������ ������� API�� ȣ���ϴ°��� �����ϴٴ� �� �Դϴ�.<br>
���� ������Ʈ���� �ɼ��� �����Ͽ� �ٷ� C# �ڵ� ������ ������ ���·� ����°� ���� �����մϴٸ�, �̺κ��� ��û�� ������ �����ϴ�.
```
/clr �ɼ��� ������ ���, ��κ��� �ٸ� ������ �ɼ��� ������� ���մϴ�.
������ c++ stdlib�� ������ ���� �κ��� ���� ������� ���Ѵٴ� ���� ���� Ů�ϴ�.

����)
<mutex> is not supported when compiling with /clr or /clr:pure.
```
���� ���־��ִ�κ��� ���� C# API�� C++ ���·� ������ DLL�� �����, �ش� DLL�� ���� ������Ʈ�� ��ũ�Ͽ� ����ϴ� ����� ����˴ϴ�.<br>
<br>
C++/CLI�� ������� �� �Ϲ����� .Net �Լ� ȣ���� ������ ���� ����˴ϴ�.
```cpp
void main() {
  System::Console::WriteLine("Hello World!");
}
```

* �ݴ�� ���� C++ �ڵ带 CLR���� ���� ������ ���̳ʸ��� ����Ͽ� C#���� C++ �ڵ带 ����� ���� �ֽ��ϴ�. ������ �̴� �ܼ��� C#���� DLL Import�� �ذ��� �� ������, ���⼭�� �� ����� ���ؼ��� �ڼ��� �ٷ��� �ʰڽ��ϴ�.


�����Ǵ� Ŭ���� �����
----
Ŭ���� ���� �տ� `ref` Ű���带 �ٿ� �����Ǵ� Ŭ������ ������ �� �ֽ��ϴ�.<br>
�����Ǵ� Ŭ������ �ٸ� �����Ǵ� Ŭ������ �ν��Ͻ��� ������Ƽ�� ������ �� �ֽ��ϴ�.
```cpp
ref class ManagedFoo {
public:
	int var = 0;
  
  // C# �� �����ϰ� ���� �������� �ʾƵ� System::Object���� ��ӹ޽��ϴ�.
  // ���� ToString �� �������̵� �ϴ� �� ���� �����մϴ�.
	System::String ^ToString() override {
		return "MY CUSTOM TOSTRING()";
	}
};
```
�����Ǵ� Ŭ������ GC�� ���� �����ǹǷ� ������ `delete` ������ �ʿ����� �ʽ��ϴ�.<br>
�Ʒ� �ڵ�� �����Ǵ� `ref` Ŭ������ �����ϴ� ����Դϴ�.
```cpp
// new ��� gcnew Ű����� �����մϴ�.
auto foo = gcnew ManagedFoo();

Console::WriteLine(foo);
```

* `!dtor`���� `ref` Ŭ������ ���� �ڼ��� �׸���� �� �������� �ٷ���� �ϴ� ������ ���� ������ �����Ƿ� �����մϴ�.


�ͼ���
----
C#/.NetAPI ������ �ͼ������� ���¸� ��ȯ�ϴ°��� �Ϲ����Դϴ�.<br>
�ͼ����� ��ȯ�� �� �ִ� �����Ǵ� �޼ҵ带 ȣ���� ���� �Ʒ��� ���� `try~catch` ������ ����մϴ�.
```cpp
try {
  auto dic = gcnew System::Collections::Generic::Dictionary<int, int>();
  
  // �������� �ʴ� Ű �˻�
  int value = dic[1234];
}
catch (Exception ^e) {
	Console::WriteLine(e);
}
```

���ڿ� ��ȯ�ϱ�
----
__System::String�� std::string ���� ��ȯ�ϱ�__
```cpp
#include <msclr/marshal_cppstd.h>

System::String ^a = "Hello World!";

// cpp string�� ��ȯ�˴ϴ�.
msclr::interop::marshal_as<std::string>(x);
```

__std::string�� System::String ���� ��ȯ�ϱ�__
```cpp
std::string str = "Hello World!";

// managed string�� ��ȯ�˴ϴ�.
gcnew System(str.c_str());
```

Action �ν��Ͻ� �����ϱ�
----
```cpp
static void TestCallback(int data) {
  printf("Hello : %d\n", data);
}

SomeFunc(gcnew System::Action<int>(&TestCallback));
```

���� �Լ� ����ϱ�
----
�Ϲ������� Action�� ������ �� �Լ� �����ʹ� �Ѱ��� �� ������ ���ٽ��� �Ѱ��� �� �����ϴ�.<br>
```cpp
gcnew System::Action<int>([](int data) {
  printf("Hello : %d\n", data);
});
```
������ �̹� ���ÿ����÷ο쿡 �Ϻ��� �ذ�å�� �����մϴ�.<br>
[StackOverflow](http://stackoverflow.com/a/26552573)<br>
�� �ڵ尡 ��� �渶������ �̷���� �ִ����� [C++ ������](https://github.com/jwvg0425) ���� ���� �ּ���.
```cpp
SomeFunc(Lambda2Delegate<>() = [](int data) {
  printf("Hello : %d\n", data);
});
```

�����Ǵ� �ν��Ͻ��� �����ϴ� C++ Ŭ���� �����
----
��Ģ�����δ� `ref`�� �ƴ� �Ϲ� Ŭ������ �����Ǵ� ������Ƽ�� ������ c++ �������̽��� ���� �� �����ϴ�.<br>
������ �Ʒ��� ���� ��ȸ�θ� �̿��Ͽ� �������̽��󿡴� ���ڵǴ� ������Ƽ�� ���������� �����Ű�� ������, �����δ� ������ �ִ� ���·� �ۼ��� �� �ֽ��ϴ�.
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

�����Ǵ� �ν��Ͻ��� �ְ�޴� �޼ҵ� �ۼ��ϱ�
----
���������� �������� ����Ǵ� �������̽� �󿡴� �����Ǵ� ������Ʈ�� �ְ�޵��� �ۼ��ϸ� �ȵǰ�����, ���������� Ŭ���������� �����Ǵ� ��ü�� �ְ�������� �и��� ���� ���Դϴ�.<br>
�ذ�å �� �ϳ��� ��ó�� ���漱���� �̿��� �ļ��� ����ϴ°Ͱ�, ��쿡 ���󼭴� �� �����ϰ� �ذ��� �� �ִ� `reinterpret_cast`�� �̿��� ����� �ֽ��ϴ�.<br>
�Ʒ������� `reinterpret_cast`�� �̿��� ����� ���� �����մϴ�.
<br>
```cpp
class Doll {
  // TODO : DollFactory�� ������ �� �ֵ��� friend�� �����մϴ�. �Ǵ� �˾Ƽ�
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
void *�� �����Ǵ� �ν��Ͻ��� �ѱ� ���� GC�� ���� ���۷��� ���� �ʴ� ���� �����ϼ���.


Ÿ�� �����ӿ�ũ ���� �����ϱ�
----
������ �𸣰����� C#������ ���� Ÿ�� .Net �����ӿ�ũ ������ �����ϴ� UI�� �������� �ʽ��ϴ�.<br>
�Ʒ� ������ ���� �������� ������ �����մϴ�.<br>
* ��� ������Ʈ -> ��Ŭ�� -> ������Ʈ ��ε�
* ���� .vcxproj
* `TargetFrameworkVersion` ���� ���ϴ� ������ ����
  * ��) v4.6
* ������Ʈ -> ��Ŭ�� ������Ʈ �ٽ� �ε�