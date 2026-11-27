# 5부. 최신 기능 + 부록

[← 4부 심화](04-advanced.md) · [목차](README.md)

## 0. 5부 안내

C#은 매년 11월 .NET과 함께 새 버전이 나온다. 교재나 인터넷 자료는 예전 버전 기준인 경우가 많아서, **"같은 일을 하는 새 문법"을 알아보는 것**이 5부의 목적이다. 예전 방식도 여전히 동작하니 외울 필요는 없고, 읽을 수만 있으면 된다.

| C# | .NET | 출시 | 대표 기능 |
| --- | --- | --- | --- |
| C# 15 | .NET 11 (STS, 2년 지원) | 2026년 11월 | 공용체 타입, 닫힌 계층, 레이블 break/continue |
| C# 14 | .NET 10 (LTS, 3년 지원) | 2025년 11월 | extension 블록, `field` 키워드 |
| C# 13 | .NET 9 | 2024년 11월 | params 컬렉션, `Lock` 타입 |
| C# 12 | .NET 8 (LTS) | 2023년 11월 | 기본 생성자, 컬렉션 식 |

- 프로젝트가 어느 .NET을 대상으로 하느냐에 따라 기본 C# 버전이 정해진다. .NET 11 프로젝트는 C# 15가 기본이다.
- 오래 지원받아야 하는 회사 프로젝트는 LTS인 .NET 10(C# 14)을 많이 쓴다. 학교 수업은 더 예전 버전일 수도 있으니 확인한다.

## 31장. C# 12: 기본 생성자와 컬렉션 식

C# 12의 두 기능은 이미 이 정리의 예제에서 자주 썼다. 여기서는 예전 방식과 나란히 놓고 비교한다.

**기본 생성자 (Primary Constructor)**: 클래스 이름 바로 옆에 생성자 매개변수를 쓴다.

```csharp
// 예전 방식
class OrderService
{
    private readonly INotifier notifier;
    public OrderService(INotifier notifier) { this.notifier = notifier; }
}

// C# 12
class OrderService(INotifier notifier)
{
    public void Order() => notifier.Send("주문 완료");   // 매개변수를 클래스 전체에서 사용
}
```

- 30장의 의존성 주입처럼 "받아서 저장만 하는" 생성자를 줄이는 데 특히 좋다.
- record의 기본 생성자(13장)와 달리, class의 기본 생성자는 자동으로 공개 속성을 만들지 않는다.

**컬렉션 식 (Collection Expression)**: 대괄호 `[ ]` 하나로 배열, List, Span 등을 만든다.

```csharp
// 예전 방식
int[] a = new int[] { 1, 2, 3 };
List<int> b = new List<int> { 1, 2, 3 };

// C# 12
int[] a2 = [1, 2, 3];
List<int> b2 = [1, 2, 3];
List<int> empty = [];

int[] merged = [.. a2, .. b2, 4];   // .. 은 펼치기(spread): [1,2,3,1,2,3,4]
```

**그 밖의 C# 12 기능**: 람다 매개변수의 기본값(`(int x = 1) => x`), 모든 타입에 별칭 붙이기(`using Point = (int X, int Y);`) 등이 있다.

## 32장. C# 13~14

C# 13과 14는 반복되는 코드를 줄이는 작은 개선이 많다. 그중 자주 보게 될 것만 골랐다.

**C# 14: field 키워드**

속성에 검사 로직을 넣으려면 예전에는 필드를 따로 만들어야 했다(8장). `field`는 컴파일러가 만든 숨은 필드를 가리킨다.

```csharp
// 예전 방식
private int score;
public int Score
{
    get => score;
    set => score = value < 0 ? 0 : value;
}

// C# 14
public int Score
{
    get;
    set => field = value < 0 ? 0 : value;   // field = 숨은 필드
}
```

**C# 14: extension 블록**: 21장에서 다뤘다. 확장 메서드뿐 아니라 확장 속성도 만들 수 있다.

**C# 14: null 조건부 대입**

```csharp
// 예전 방식
if (customer != null) customer.Order = GetOrder();

// C# 14: customer가 null이면 대입하지 않고 넘어감
customer?.Order = GetOrder();
```

**C# 13 주요 기능**

| 기능 | 설명 |
| --- | --- |
| params 컬렉션 | `params`에 배열뿐 아니라 `List<T>`, `Span<T>` 등도 쓸 수 있음 |
| `Lock` 타입 | `lock` 전용 타입 (24장) |
| partial 속성 | 속성도 선언과 구현을 나눌 수 있음 (소스 생성기용, 28장) |

```csharp
int Sum(params ReadOnlySpan<int> nums)   // C# 13: Span으로 받아 배열 할당 없음
{
    int total = 0;
    foreach (var n in nums) total += n;
    return total;
}
Console.WriteLine(Sum(1, 2, 3));   // 6
```

## 33장. C# 15 (2026년 11월)

C# 15의 가장 큰 변화는 공용체 타입(union)이다. "이 값은 A, B, C 중 하나"를 타입으로 표현하고, 컴파일러가 빠진 경우가 없는지 검사해 준다. 아래 예제는 Microsoft 공식 문서의 예제를 바탕으로 했다.

**공용체 타입 (Union Types)**

```csharp
public record class Cat(string Name);
public record class Dog(string Name);
public record class Bird(string Name);

public union Pet(Cat, Dog, Bird);       // Pet은 Cat, Dog, Bird 중 하나

Pet pet = new Dog("Rex");               // 각 타입에서 자동 변환

string name = pet switch
{
    Dog d => d.Name,
    Cat c => c.Name,
    Bird b => b.Name,                    // 하나라도 빠지면 컴파일러가 알려줌
};
```

2부의 인터페이스나 상속과 달리, 서로 관계없는 타입들을 묶을 수 있고 `_` 없이도 모든 경우를 다뤘는지 확인된다. "성공 또는 실패" 같은 결과를 표현할 때 유용하다.

**닫힌 계층 (Closed Hierarchies)**

`closed`를 붙인 클래스는 같은 프로젝트 안에서만 상속할 수 있다. 자식이 모두 알려져 있으므로 switch에 `_`가 필요 없다.

```csharp
public closed record class GateState;          // closed는 자동으로 abstract
public record class Closed : GateState;
public record class Open(float Percent) : GateState;

string Describe(GateState state) => state switch
{
    Closed => "닫힘",
    Open(var percent) => $"{percent}% 열림",
    // 경고 없음: 모든 자식을 다루었음
};
```

**레이블이 붙은 break / continue**

중첩 반복문(4장)에서 바깥 반복문까지 한 번에 빠져나갈 수 있다. 예전에는 bool 변수나 `goto`를 써야 했다.

```csharp
outer: for (int row = 0; row < 5; row++)
{
    for (int col = 0; col < 5; col++)
    {
        if (grid[row, col] == 0) continue outer;   // 다음 row로
        if (grid[row, col] == 9) break outer;      // 바깥 반복문까지 종료
    }
}
```

**그 밖의 C# 15 기능**

| 기능 | 설명 |
| --- | --- |
| 컬렉션 식 인수 | `[with(capacity: 100), .. items]`처럼 생성자 인수를 넘길 수 있음 |
| 확장 인덱서 | extension 블록에 인덱서도 선언 가능 (21장) |
| 메모리 안전성 (미리보기) | unsafe 규칙 재설계. 유일하게 preview 언어 버전이 필요 (29장) |

```csharp
HashSet<string> set = [with(StringComparer.OrdinalIgnoreCase), "Hello", "HELLO"];
// 대소문자를 무시하므로 요소는 1개
```

## 부록 A. 단위 테스트 (xUnit)

단위 테스트는 **메서드가 제대로 동작하는지 확인하는 코드를 따로 작성해서 자동으로 검사**하는 것이다. 코드를 고칠 때마다 콘솔에 직접 입력하며 확인하던 일을 명령 하나로 끝낸다. C#에서는 xUnit이 가장 널리 쓰인다.

**준비**

```bash
dotnet new xunit -o MyApp.Tests                 # 테스트 프로젝트 생성
dotnet add MyApp.Tests reference MyApp          # 테스트할 프로젝트 연결
dotnet test                                     # 실행
```

**테스트 작성: 준비 → 실행 → 확인 (AAA)**

```csharp
public class BankAccountTests
{
    [Fact]                                       // 테스트 하나
    public void Withdraw_MoreThanBalance_ReturnsFalse()
    {
        // Arrange (준비)
        var account = new BankAccount("철수");
        account.Deposit(1000);

        // Act (실행)
        bool result = account.Withdraw(5000);

        // Assert (확인)
        Assert.False(result);
        Assert.Equal(1000m, account.Balance);
    }

    [Theory]                                     // 값을 바꿔가며 여러 번
    [InlineData(95, "A")]
    [InlineData(85, "B")]
    [InlineData(50, "C 이하")]
    public void Grade_ReturnsExpected(int score, string expected)
        => Assert.Equal(expected, Grader.Grade(score));

    [Fact]
    public void Deposit_Negative_Throws()
        => Assert.Throws<ArgumentException>(() => new BankAccount("영희").Deposit(-1));
}
```

- 테스트 이름은 `메서드_상황_기대결과`처럼 쓰면 실패했을 때 무엇이 틀렸는지 바로 보인다.
- 경계값(0, 음수, 빈 문자열, 최댓값)을 테스트하면 버그를 많이 잡을 수 있다.
- 30장의 의존성 주입을 쓰면 이메일·파일·네트워크를 가짜로 바꿔 끼워 테스트할 수 있다. DI와 테스트가 짝으로 소개되는 이유이다.

**연습**: 1부부터 만든 `IsPrime`, `Grade`, `BankAccount`에 테스트를 각각 3개 이상 작성한다.

## 부록 B. 실제 앱 맛보기와 다음 방향

C# 문법을 배운 다음에는 만들고 싶은 것에 맞는 프레임워크를 고른다. 모두 `dotnet new` 한 줄로 시작할 수 있다.

| 만들고 싶은 것 | 프레임워크 | 시작 명령 |
| --- | --- | --- |
| 콘솔 도구 | 콘솔 앱 | `dotnet new console` |
| 웹 서버, REST API | ASP.NET Core | `dotnet new webapi` |
| 웹 화면(C#만으로) | Blazor | `dotnet new blazor` |
| 모바일·데스크톱 앱 | .NET MAUI | `dotnet new maui` |
| Windows 데스크톱 | WPF, WinForms | `dotnet new wpf` |
| 게임 | Unity (별도 설치) | Unity Hub에서 생성 |
| 데이터베이스 | Entity Framework Core | NuGet 패키지 추가 |

**ASP.NET Core 맛보기: API 한 개**

```csharp
// dotnet new web 으로 만든 프로젝트의 Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<INotifier, EmailNotifier>();   // 30장 DI 그대로
var app = builder.Build();

List<Student> students = [new("철수", 85), new("영희", 92)];

app.MapGet("/students", () => students);                                  // 목록 (JSON으로 자동 변환)
app.MapGet("/students/top", () => students.MaxBy(s => s.Score));          // 18장 LINQ

app.Run();   // 브라우저에서 http://localhost:포트/students 로 확인

record Student(string Name, int Score);
```

1~4부에서 배운 것(record, 람다, LINQ, JSON, DI)이 그대로 쓰인다는 점에 주목한다.

**다음 공부 방향**

- **웹 개발**: ASP.NET Core → EF Core(데이터베이스) → 인증 → 배포
- **게임 개발**: Unity로 간단한 2D 게임 만들기. 17장 이벤트와 2부 객체지향이 핵심
- **앱 개발**: .NET MAUI로 안드로이드·iOS 앱 만들기
- **기초 다지기**: 자료구조와 알고리즘 문제를 C#으로 풀기(백준, 프로그래머스)

어느 쪽이든 **작은 프로젝트를 끝까지 완성해 보는 것**이 문법을 더 읽는 것보다 훨씬 빨리 능숙해지는 길이다.

## 참고 자료

- [What's new in C# 15 (Microsoft Learn)](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-15) — 33장 예제의 출처
- [What's new in C# (버전별 목록)](https://learn.microsoft.com/dotnet/csharp/whats-new)
- [C# 공식 문서 (한국어)](https://learn.microsoft.com/ko-kr/dotnet/csharp/) — 기초 문법을 찾아볼 때 가장 정확한 자료

1~4부는 따로 참고한 문서 없이 일반적인 C# 지식으로 작성했다. 헷갈리는 부분은 위 공식 문서와 대조해 본다.

---

[← 4부 심화](04-advanced.md) · [목차](README.md)
