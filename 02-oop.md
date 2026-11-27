# 2부. 객체지향

[← 1부 기초](01-basics.md) · [목차](README.md) · [3부 중급 →](03-intermediate.md)

## 0. 객체지향이란

객체지향은 프로그램을 **데이터와 그 데이터를 다루는 기능을 한 덩어리(객체)로 묶어서** 만드는 방식이다. C#은 객체지향 언어라서, 2부부터 배우는 내용이 실제 C# 코드의 대부분을 차지한다.

1부의 성적 관리기를 떠올려 본다. `이름 배열`과 `점수 배열`을 따로 두면 순서가 어긋나기 쉽고, 항목(학번, 학과)이 늘 때마다 배열도 늘어난다. 객체지향에서는 `Student`라는 틀을 만들고 학생 한 명을 객체 하나로 다룬다.

**객체지향의 네 가지 핵심**

| 개념 | 한 줄 설명 | 배우는 곳 |
| --- | --- | --- |
| 캡슐화 | 데이터를 숨기고 정해진 방법으로만 만지게 한다 | 9장 |
| 상속 | 기존 클래스를 물려받아 확장한다 | 10장 |
| 다형성 | 같은 명령에 객체마다 다르게 반응한다 | 10장 |
| 추상화 | 공통된 꼴만 정하고 세부는 감춘다 | 10~11장 |

**공부 순서**: 8장(클래스)이 모든 것의 바탕이니 가장 오래 본다. 9~11장은 한 묶음으로 이어 보고, 12~14장은 조금 가벼운 보충 주제이다. 단, 12장의 값 타입과 참조 타입은 시험에 자주 나오는 중요한 개념이다.

예제는 1부처럼 최상위 문 방식이다. 실행 코드를 위에, 클래스 선언을 파일 아래쪽에 두면 된다.

## 8장. 클래스와 객체

클래스는 객체를 만들기 위한 설계도(틀)이고, 객체(인스턴스)는 그 설계도로 **실제 만든 물건**이다. 붕어빵 틀이 클래스, 구워진 붕어빵 하나하나가 객체라고 생각하면 된다.

```csharp
// 사용하는 쪽 (파일 위쪽)
Student s1 = new Student("홍길동", 85);  // new로 객체 생성
var s2 = new Student("김철수", 92);
Student s3 = new("이영희", 78);       // 타입이 보이면 new()만 써도 됨

s1.Introduce();                      // 저는 홍길동, 85점이다
Console.WriteLine(s2.Name);          // 김철수

// 설계도 (파일 아래쪽)
class Student
{
    // 필드: 객체가 갖는 데이터
    private int score;

    // 속성: 필드에 접근하는 창구
    public string Name { get; set; }
    public int Score
    {
        get { return score; }
        set { score = value < 0 ? 0 : value; }   // 음수는 0으로
    }

    // 생성자: new 할 때 딱 한 번 실행
    public Student(string name, int score)
    {
        Name = name;
        Score = score;
    }

    // 메서드: 객체가 하는 동작
    public void Introduce() => Console.WriteLine($"저는 {Name}, {Score}점입니다");
}
```

**클래스를 이루는 것들 (멤버)**

| 멤버 | 역할 | 예 |
| --- | --- | --- |
| 필드 | 객체 안에 저장되는 변수 | `private int score;` |
| 속성 | 필드를 읽고 쓰는 창구, 검사 로직을 넣을 수 있음 | `public string Name { get; set; }` |
| 생성자 | 객체를 만들 때 초기값을 정함, 클래스와 이름이 같고 반환형이 없음 | `public Student(...)` |
| 메서드 | 객체가 하는 동작 | `Introduce()` |

**알아두면 좋은 점**

- **자동 구현 속성**: `{ get; set; }`만 쓰면 컴파일러가 숨은 필드를 만들어 준다. 실무에서는 대부분 이 형태를 쓴다.
- **읽기 전용**: `{ get; }`만 있으면 생성자에서만 값을 정할 수 있다. `{ get; init; }`은 객체를 만들 때까지만 값을 줄 수 있다.
- **this**: 객체 자기 자신을 가리킨다. 매개변수와 필드 이름이 같을 때 `this.score = score;`처럼 구분한다.
- **생성자 오버로딩**: 생성자도 여러 개 만들 수 있다. 생성자를 하나도 안 쓰면 매개변수 없는 기본 생성자가 자동으로 생긴다.
- **객체 이니셜라이저**: `new Student("홍길동", 0) { Score = 90 }`처럼 만들면서 속성을 바로 채울 수 있다.
- **기본 생성자(primary constructor)**: 최신 문법으로 `class Point(int x, int y) { ... }`처럼 클래스 이름 옆에 바로 매개변수를 쓸 수도 있다. 교재에는 잘 안 나오니 보면 알아보는 정도면 된다.

**연습**: `BankAccount` 클래스를 만든다. 예금주 이름과 잔액을 갖고, `Deposit(금액)`과 `Withdraw(금액)` 메서드를 넣는다. 잔액보다 많이 출금하면 거절해야 한다.

## 9장. 캡슐화와 접근 제한자

캡슐화는 객체의 데이터를 밖에서 마음대로 바꾸지 못하게 숨기고, 정해진 메서드와 속성으로만 다루게 하는 것이다. 자판기를 생각하면 쉽다. 안의 돈통은 못 만지고 버튼과 동전 투입구만 쓸 수 있다.

**왜 필요한가**

```csharp
// 캡슐화가 없으면
account.Balance = -1000000;   // 누구나 잔액을 이상하게 바꿀 수 있음

// 캡슐화를 하면
account.Withdraw(1000);       // 메서드 안에서 잔액을 검사한 뒤에만 바뀜
```

**접근 제한자**

| 제한자 | 접근 가능한 곳 |
| --- | --- |
| `public` | 어디서나 |
| `private` | 같은 클래스 안에서만 (클래스 멤버의 기본값) |
| `protected` | 같은 클래스와 상속받은 자식 클래스 |
| `internal` | 같은 프로젝트(어셈블리) 안에서 (클래스 자체의 기본값) |
| `protected internal` | 같은 프로젝트 또는 자식 클래스 |
| `private protected` | 같은 프로젝트 안의 자식 클래스 |

처음에는 위의 세 개(`public`, `private`, `protected`)만 확실히 알면 된다.

```csharp
class BankAccount
{
    public string Owner { get; }
    public decimal Balance { get; private set; }  // 읽기는 누구나, 쓰기는 클래스 안에서만

    public BankAccount(string owner) => Owner = owner;

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentException("입금액은 0보다 커야 합니다");
        Balance += amount;
    }

    public bool Withdraw(decimal amount)
    {
        if (amount > Balance) return false;   // 잔액 부족
        Balance -= amount;
        return true;
    }
}
```

**원칙**

- 필드는 `private`로 두고, 밖에 보여줄 것만 속성이나 메서드로 `public`으로 연다.
- `{ get; private set; }`은 "밖에서는 읽기만" 하게 하는 가장 흔한 패턴이다.
- 접근 제한자를 생략하면 멤버는 `private`이다. 밖에서 안 보인다면 `public`을 빠뜨린 건 아닌지 먼저 확인한다.

**연습**: 8장에서 만든 `BankAccount`를 위 원칙대로 고치고, `account.Balance = 100;`이 컴파일 오류가 나는지 확인해 본다.

## 10장. 상속과 다형성

상속은 기존 클래스(부모)의 멤버를 물려받아 새 클래스(자식)를 만드는 것이고, 다형성은 부모 타입으로 묶인 객체들이 같은 메서드 호출에 각자 다르게 반응하는 것이다. "개는 동물이다(is-a)"처럼 말이 되는 관계에만 상속을 쓴다.

```csharp
Animal[] animals = [new Dog("바둑이"), new Cat("나비")];
foreach (Animal a in animals)
    a.Speak();          // 바둑이: 멍멍! / 나비: 야옹~  ← 다형성

class Animal
{
    public string Name { get; }
    public Animal(string name) => Name = name;

    public virtual void Speak()             // virtual: 자식이 바꿀 수 있음
        => Console.WriteLine($"{Name}: ...");
}

class Dog : Animal                          // : 부모 = 상속
{
    public Dog(string name) : base(name) { } // base: 부모 생성자 호출
    public override void Speak()             // override: 다시 정의
        => Console.WriteLine($"{Name}: 멍멍!");
}

class Cat : Animal
{
    public Cat(string name) : base(name) { }
    public override void Speak() => Console.WriteLine($"{Name}: 야옹~");
}
```

**핵심 키워드**

| 키워드 | 의미 |
| --- | --- |
| `class 자식 : 부모` | 부모를 상속한다. C# 클래스는 부모를 **하나만** 가질 수 있다 |
| `base` | 부모를 가리킨다. `base(...)`는 부모 생성자, `base.Speak()`는 부모의 메서드 |
| `virtual` | "자식이 이 메서드를 바꿔도 된다" |
| `override` | 부모의 virtual/abstract 메서드를 다시 정의한다 |
| `abstract` | 몸체 없이 꼴만 있는 메서드. 자식이 반드시 구현해야 한다 |
| `sealed` | 더 이상 상속하거나 override할 수 없게 막는다 |

**추상 클래스**

"도형"처럼 그 자체로는 만들 수 없고 자식(원, 사각형)을 통해서만 쓰는 클래스이다.

```csharp
abstract class Shape
{
    public abstract double Area();          // 몸체 없음
    public void Print() => Console.WriteLine($"넓이: {Area():F2}");
}

class Circle(double r) : Shape
{
    public override double Area() => Math.PI * r * r;
}

class Rect(double w, double h) : Shape
{
    public override double Area() => w * h;
}

// new Shape();  ← 오류: 추상 클래스는 직접 만들 수 없음
```

**헷갈리는 점**

- **오버로딩 vs 오버라이딩**: 오버로딩은 같은 이름에 매개변수가 다른 메서드를 여러 개 만드는 것(6장), 오버라이딩은 부모의 메서드를 자식이 다시 정의하는 것이다.
- **형변환**: 자식→부모는 자동(`Animal a = new Dog(...)`)이다. 부모→자식은 확인이 필요하다. `if (a is Dog d) { ... }`로 검사하면서 변환한다.
- 모든 클래스의 조상은 `object`이다. `ToString()`, `Equals()`를 override해서 자주 쓴다.
- 상속을 너무 깊게(3단계 이상) 만들면 코드를 이해하기 어려워진다. "가지고 있다(has-a)" 관계는 상속 대신 필드로 담는다(자동차는 엔진을 가진다).

**연습**: `Employee`(이름, `abstract decimal GetPay()`)를 만들고, `FullTime`(월급)과 `PartTime`(시급 × 시간)이 상속받게 한다. `Employee[]` 배열 하나로 전체 급여 합계를 구한다.

## 11장. 인터페이스

인터페이스는 "이 기능들을 반드시 갖춰야 한다"는 약속(계약서)이다. 상속이 "~이다(is-a)"라면 인터페이스는 "~할 수 있다(can-do)"이다. 클래스는 부모를 하나만 가질 수 있지만, 인터페이스는 여러 개를 구현할 수 있다.

```csharp
IFlyable[] flyers = [new Bird(), new Drone()];
foreach (var f in flyers)
    f.Fly();       // 새와 드론은 전혀 다른 종류지만 둘 다 날 수 있음

interface IFlyable          // 이름은 보통 I로 시작
{
    void Fly();             // 몸체 없이 꼴만. public이 기본
}

interface ISwimmable { void Swim(); }

class Bird : IFlyable
{
    public void Fly() => Console.WriteLine("날갯짓");
}

class Drone : IFlyable
{
    public void Fly() => Console.WriteLine("프로펠러 회전");
}

class Duck : Animal, IFlyable, ISwimmable   // 부모 하나 + 인터페이스 여러 개
{
    public Duck() : base("오리") { }
    public void Fly() => Console.WriteLine("파닥파닥");
    public void Swim() => Console.WriteLine("둥둥");
}
```

**추상 클래스와 비교**

| | 추상 클래스 | 인터페이스 |
| --- | --- | --- |
| 관계 | ~이다 (is-a) | ~할 수 있다 (can-do) |
| 여러 개 가능? | 하나만 상속 | 여러 개 구현 |
| 필드(상태) | 가질 수 있음 | 가질 수 없음 |
| 생성자 | 있음 | 없음 |
| 언제 쓰나 | 가까운 클래스들이 코드를 공유할 때 | 서로 다른 클래스에 같은 능력을 줄 때 |

**알아두면 좋은 점**

- 인터페이스를 구현하면 그 안의 멤버를 **빠짐없이** 만들어야 한다. 하나라도 빠지면 컴파일 오류가 난다.
- .NET에는 이미 만들어진 인터페이스가 많다. `IComparable`(정렬 기준), `IDisposable`(자원 정리), `IEnumerable`(foreach로 돌 수 있음)이 대표적이다.
- 인터페이스 타입으로 변수를 만들면 나중에 구현 클래스를 바꿔 끼우기 쉽다. 이 성질이 4부의 의존성 주입으로 이어진다.
- 최신 C#에서는 인터페이스에도 기본 구현(몸체 있는 메서드)을 넣을 수 있지만, 처음에는 "꼴만 있는 약속"으로 이해하면 충분하다.

**연습**: `Student`에 `IComparable<Student>`를 구현해서(`CompareTo` 메서드), `Array.Sort(students)`를 하면 점수 높은 순으로 정렬되게 만든다.

## 12장. 값 타입과 참조 타입 (struct와 class)

**값 타입은 변수에 값 자체가 들어 있고, 참조 타입은 변수에 객체가 있는 위치(주소)가 들어 있다.** 그래서 대입하거나 메서드에 넘길 때, 값 타입은 복사본이 생기고 참조 타입은 같은 객체를 함께 가리키게 된다.

| | 값 타입 | 참조 타입 |
| --- | --- | --- |
| 예 | `int`, `double`, `bool`, `char`, `struct`, `enum` | `class`, `string`, 배열, `interface`, `record` |
| 변수에 들어 있는 것 | 값 자체 | 객체의 주소 |
| 대입하면 | 값이 통째로 복사됨 | 주소만 복사됨 (같은 객체) |
| `null` | 될 수 없음 (`int?`는 가능) | 될 수 있음 |
| 주로 저장되는 곳 | 스택 | 힙 (가비지 컬렉터가 정리) |

```csharp
// struct (값 타입)
PointS a = new PointS { X = 1 };
PointS b = a;        // 복사본
b.X = 99;
Console.WriteLine(a.X);   // 1  ← a는 그대로

// class (참조 타입)
PointC c = new PointC { X = 1 };
PointC d = c;        // 같은 객체를 가리킴
d.X = 99;
Console.WriteLine(c.X);   // 99 ← c도 바뀜

struct PointS { public int X; }
class  PointC { public int X; }
```

```
값 타입 (struct)                 참조 타입 (class)
스택                             스택            힙
┌────────┐ ┌────────┐            ┌──────────┐    ┌─────────────┐
│ a      │ │ b      │            │ c : 주소 │───▶│ PointC 객체 │
│ X = 1  │ │ X = 99 │            ├──────────┤    │ X = 99      │
└────────┘ └────────┘            │ d : 주소 │───▶│             │
 서로 다른 상자                  └──────────┘    └─────────────┘
                                  주소만 복사, 객체는 하나
```

**꼭 알아야 할 점**

- **배열은 참조 타입**이다. 배열을 메서드에 넘기고 안에서 요소를 바꾸면 원본도 바뀐다. 6장의 "값 복사 전달"은 주소가 복사된다는 뜻이다.
- **`string`은 참조 타입이지만 불변**이라 값 타입처럼 느껴진다. 바꾸는 순간 새 문자열이 생기기 때문이다.
- **`null`**은 "가리키는 객체가 없음"이다. `null`인 변수의 멤버를 쓰면 `NullReferenceException`이 난다.
- **struct는 언제 쓰나**: 좌표, 색상처럼 작고 값 자체가 의미인 데이터에 쓴다. 잘 모르겠으면 class를 쓴다.
- **박싱/언박싱**: 값 타입을 `object`에 담으면 힙에 상자를 만들어 넣는다(박싱). 느려질 수 있어서 3부의 제네릭이 이 문제를 해결한다.

**연습**: `int[]`를 받아 첫 요소를 0으로 바꾸는 메서드와 `int`를 받아 0으로 바꾸는 메서드를 만들고, 호출 후 원본이 어떻게 되는지 예측한 다음 실행해 확인한다.

## 13장. enum, record, 튜플

세 가지 모두 "클래스를 따로 만들기엔 과한" 데이터를 간단하게 표현하는 도구이다. enum은 정해진 선택지, record는 데이터 묶음, 튜플은 일회용 묶음에 쓴다.

**enum (열거형)**: 값이 정해진 몇 개 중 하나일 때 쓴다. 숫자나 문자열 대신 쓰면 오타가 컴파일 오류로 잡힌다.

```csharp
OrderStatus s = OrderStatus.Shipped;
if (s == OrderStatus.Shipped) Console.WriteLine("배송 중");
Console.WriteLine((int)s);       // 1 (내부적으로는 정수)
Console.WriteLine(s);            // Shipped

enum OrderStatus { Ordered, Shipped, Delivered, Canceled }  // 0, 1, 2, 3
```

**record**: 데이터를 담는 것이 목적인 타입이다. 한 줄로 만들 수 있고, 내용이 같으면 같다고 비교되며, 출력하면 내용이 보인다.

```csharp
var p1 = new Person("홍길동", 20);
var p2 = new Person("홍길동", 20);
Console.WriteLine(p1 == p2);     // True  (class였다면 False)
Console.WriteLine(p1);           // Person { Name = 홍길동, Age = 20 }

var older = p1 with { Age = 21 }; // 일부만 바꾼 복사본 (원본은 그대로)

record Person(string Name, int Age);
```

- record는 기본적으로 값을 바꿀 수 없다(불변). 바꾸려면 `with`로 새로 만든다.
- class는 "같은 객체인가"를, record는 "내용이 같은가"를 비교한다.

**튜플**: 이름을 붙일 필요 없는 값 몇 개를 잠깐 묶을 때 쓴다. 메서드에서 값을 여러 개 돌려줄 때 특히 편하다.

```csharp
(int Min, int Max) MinMax(int[] arr) => (arr.Min(), arr.Max());

var r = MinMax([3, 9, 1]);
Console.WriteLine($"{r.Min} ~ {r.Max}");   // 1 ~ 9

var (lo, hi) = MinMax([3, 9, 1]);           // 분해해서 받기
```

6장의 `out` 매개변수 대신 튜플을 쓰면 코드가 더 깔끔해진다.

| 상황 | 쓸 것 |
| --- | --- |
| 선택지가 정해져 있음 (요일, 상태, 등급) | enum |
| 데이터를 주고받는 것이 목적 (회원 정보, 설정값) | record |
| 메서드 안에서만 잠깐 쓸 묶음 | 튜플 |
| 동작과 상태가 중요 (계좌, 게임 캐릭터) | class |

**연습**: `enum Grade { A, B, C, D, F }`를 만들고, 점수를 받아 `Grade`를 돌려주는 메서드를 switch 식으로 작성한다.

## 14장. static, partial, 중첩 클래스

`static`은 객체 하나하나가 아니라 **클래스 자체에 속하는** 멤버를 만든다. `new` 없이 `클래스이름.멤버`로 바로 쓰며, 모든 객체가 하나를 공유한다. `Console.WriteLine`, `Math.Max`가 모두 static이다.

```csharp
var a = new Student("홍길동");
var b = new Student("김철수");
Console.WriteLine(Student.Count);        // 2  ← 클래스 이름으로 접근
Console.WriteLine(MathUtil.Square(5));   // 25

class Student
{
    public static int Count { get; private set; }  // 모든 학생이 공유
    public string Name { get; }                     // 학생마다 따로

    public Student(string name) { Name = name; Count++; }
}

static class MathUtil                    // static 클래스: 객체를 만들 수 없음
{
    public static int Square(int x) => x * x;
}
```

- **인스턴스 멤버**(static 없음)는 객체마다 따로 있고, **static 멤버**는 클래스에 하나만 있다.
- static 메서드 안에서는 `this`도, 인스턴스 멤버(`Name`)도 쓸 수 없다. 어느 객체의 것인지 모르기 때문이다. 초보자가 자주 만나는 컴파일 오류이다.
- 객체의 상태가 필요 없는 계산 도구(단위 변환, 수학 함수)는 static 클래스에 모으면 좋다.

**partial**: 한 클래스를 여러 파일에 나눠 적을 수 있게 한다. 컴파일할 때 하나로 합쳐진다. WinForms나 WPF처럼 도구가 자동으로 만드는 코드와 내가 쓰는 코드를 분리할 때 주로 보게 된다.

```csharp
// Form1.Designer.cs (자동 생성)
partial class Form1 { /* 버튼 배치 코드 */ }

// Form1.cs (내가 작성)
partial class Form1 { /* 버튼 클릭 처리 코드 */ }
```

**중첩 클래스**: 클래스 안에 만든 클래스이다. 바깥 클래스에서만 쓰는 보조 타입을 숨길 때 쓴다.

```csharp
class Library
{
    private class Book { public string Title = ""; }  // Library 안에서만 사용
    private Book[] books = new Book[100];
}
```

**연습**: `Product` 클래스에 static 카운터를 두고, 객체를 만들 때마다 `P001`, `P002`처럼 자동으로 상품 코드가 붙게 한다.

## 2부 마무리

아래 미니 프로젝트를 클래스 설계부터 직접 해 보면 2부가 정리된다. 설계가 정답이 하나인 문제는 아니니, 먼저 종이에 클래스 이름과 관계를 그려 본다.

- [ ] **도서관 대출 프로그램**: `Book`(제목, 저자, 대출 여부)과 `Member`(이름, 빌린 책 목록)를 만든다. 회원은 최대 3권까지 빌릴 수 있고, 이미 대출된 책은 빌릴 수 없게 한다(캡슐화). 책 상태는 enum으로 표현한다.
- [ ] **도형 그림판**: `abstract class Shape`를 부모로 `Circle`, `Rect`, `Triangle`을 만들고, `Shape[]`에 섞어 담은 뒤 넓이 합계와 가장 큰 도형을 출력한다(상속, 다형성). 넓이 순 정렬은 `IComparable`로 해 본다.

**다음 단계: 3부 중급**

2부에서는 배열에 객체를 담았는데, 크기가 고정돼 있어 불편했을 거다. 3부에서는 제네릭과 컬렉션(`List<T>`, `Dictionary<K,V>`)으로 이 문제를 풀고, 델리게이트와 람다, LINQ로 데이터를 검색·정렬·가공하는 법을 배운다.

---

[← 1부 기초](01-basics.md) · [목차](README.md) · [3부 중급 →](03-intermediate.md)
