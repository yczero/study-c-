# 1부. C# 기초

[← 목차로](README.md) · [다음: 2부 객체지향 →](02-oop.md)

## 0. 무엇부터 해야 하나

처음이라면 개발 환경을 설치하고 Hello World를 직접 실행해 보는 것부터 시작한다. 그다음 1장부터 7장까지 순서대로 공부하면 된다. 앞 장의 내용이 뒷장의 재료가 되므로 건너뛰지 않는 게 좋다.

1. **환경 설치 (1일)**: .NET SDK와 편집기(Visual Studio 또는 VS Code)를 설치하고 Hello World를 실행한다.
2. **값 다루기 (1주)**: 변수, 자료형, 연산자(2~3장)를 배운다. 계산기 같은 간단한 프로그램을 만들 수 있게 된다.
3. **흐름 제어 (1주)**: 조건문과 반복문(4장)을 배운다. 프로그래밍의 핵심 사고방식이 여기서 생긴다.
4. **여러 값 묶기 (1주)**: 배열과 문자열(5장)을 배운다.
5. **코드 나누기 (1주)**: 메서드(6장)로 코드를 기능별로 쪼갠다.
6. **오류 대비 (2~3일)**: 예외 처리(7장)로 프로그램이 갑자기 꺼지지 않게 한다.
7. **다음 단계**: 2부 객체지향(클래스, 상속, 인터페이스)으로 넘어간다.

공부 방법은 세 가지만 지키면 된다.

- 예제 코드는 눈으로만 보지 말고 직접 타이핑해서 실행한다.
- 값을 바꿔 보고 일부러 오류도 내 보면서 결과가 어떻게 달라지는지 확인한다.
- 장마다 끝에 있는 연습문제를 꼭 직접 풀어 본다.

## 1장. C#과 .NET, 첫 프로그램

C#은 마이크로소프트가 만든 프로그래밍 언어이고, .NET은 C# 코드를 실행해 주는 플랫폼이다. 언어(C#)와 실행 환경(.NET)은 따로라는 점만 기억하면 된다.

**핵심 개념**

- **컴파일**: 사람이 쓴 코드를 컴퓨터가 실행할 수 있는 형태로 바꾸는 일이다. 문법이 틀리면 이 단계에서 오류(컴파일 오류)가 난다.
- **크로스 플랫폼**: .NET은 Windows, macOS, Linux에서 모두 동작한다.
- **활용 분야**: 게임(Unity), 웹 서버(ASP.NET Core), 데스크톱 앱, 모바일 앱(MAUI) 등에 쓰인다.
- **버전**: 2026년 11월 기준 최신은 C# 15와 .NET 11이다. 기초 문법은 버전과 관계없이 같다.

**설치와 실행**

1. .NET SDK를 설치한다.
2. 터미널에서 `dotnet new console -o HelloApp`으로 프로젝트를 만든다.
3. `cd HelloApp` 후 `dotnet run`으로 실행한다.

**Hello World**

```csharp
// 최신 방식 (최상위 문): 바로 코드를 쓴다
Console.WriteLine("Hello, World!");
```

교재나 예전 자료에서는 아래처럼 긴 형태를 자주 보게 된다. 두 코드는 같은 동작을 한다.

```csharp
using System;              // System 이름공간을 사용하겠다

namespace HelloApp         // 코드를 묶는 이름
{
    class Program          // 클래스 (2부에서 배움)
    {
        static void Main() // 프로그램이 시작하는 곳
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

- `Console.WriteLine()`: 화면에 한 줄을 출력한다. `Console.Write()`는 줄바꿈이 없다.
- `Console.ReadLine()`: 사용자가 입력한 한 줄을 문자열로 받는다.
- 문장 끝에는 세미콜론(`;`)을 붙인다.
- `//`는 한 줄 주석, `/* */`는 여러 줄 주석이다.
- C#은 대소문자를 구분한다. `console`과 `Console`은 다르다.

**연습**: 이름을 입력받아 `안녕하세요, OOO님!`을 출력해 본다.

## 2장. 변수와 자료형

변수는 값을 담아 두는 이름 붙은 상자이고, 자료형은 그 상자에 들어갈 값의 종류이다. C#은 한 번 정한 자료형을 바꿀 수 없는 언어(정적 타입)이다.

```csharp
int age = 20;           // 자료형 변수이름 = 값;
age = 21;               // 값은 바꿀 수 있음
// age = "스물하나";    // 오류: int에 문자열은 못 넣음
```

**자주 쓰는 자료형**

| 자료형 | 담는 값 | 예시 |
| --- | --- | --- |
| `int` | 정수 (약 ±21억) | `int count = 10;` |
| `long` | 큰 정수 | `long big = 10_000_000_000L;` |
| `double` | 실수 (기본) | `double pi = 3.14;` |
| `float` | 작은 실수 | `float f = 1.5f;` |
| `decimal` | 정밀한 실수 (돈 계산) | `decimal price = 9.99m;` |
| `bool` | 참/거짓 | `bool isOk = true;` |
| `char` | 문자 한 개 | `char grade = 'A';` |
| `string` | 문자열 | `string name = "철수";` |

- 문자 한 개는 작은따옴표(`'A'`), 문자열은 큰따옴표(`"A"`)이다.
- `double`은 `0.1 + 0.2`가 정확히 `0.3`이 아닐 수 있다. 돈 계산에는 `decimal`을 쓴다.

**var: 자료형 추론**

```csharp
var score = 95;       // 컴파일러가 int로 알아서 정함
var msg = "hi";       // string
```

`var`도 자료형이 정해진 변수이다. 다만 오른쪽 값을 보고 컴파일러가 대신 적어 줄 뿐이라, 처음에는 자료형을 직접 쓰며 익히는 걸 권한다.

**상수**

```csharp
const double TaxRate = 0.1;  // 한 번 정하면 바꿀 수 없음
```

**형변환**

- **암시적 변환**: 작은 형에서 큰 형으로는 자동이다. `double d = 10;`
- **명시적 변환(캐스팅)**: 값이 잘릴 수 있으면 직접 써야 한다. `int i = (int)3.9;` 결과는 `3`(반올림이 아니라 버림)이다.
- **문자열 → 숫자**: `int.Parse("123")`, 실패할 수 있으면 `int.TryParse("abc", out int n)`을 쓴다.
- **숫자 → 문자열**: `123.ToString()`

```csharp
Console.Write("나이: ");
string input = Console.ReadLine();
if (int.TryParse(input, out int age))
    Console.WriteLine($"내년에는 {age + 1}살");
else
    Console.WriteLine("숫자를 입력하세요");
```

**연습**: 섭씨 온도를 입력받아 화씨로 바꿔 출력한다. (F = C × 9 / 5 + 32)

## 3장. 연산자와 식

연산자는 값을 계산하거나 비교하는 기호이고, 식은 연산의 결과로 값 하나가 나오는 코드 조각이다. 초보자가 가장 많이 틀리는 부분은 정수 나눗셈과 `=`/`==`의 차이이다.

| 종류 | 연산자 | 예시 (결과) |
| --- | --- | --- |
| 산술 | `+ - * / %` | `7 / 2` (3), `7 % 2` (1) |
| 대입 | `= += -= *= /=` | `x += 5` (x = x + 5) |
| 증감 | `++ --` | `x++` (x에 1 더하기) |
| 비교 | `== != < > <= >=` | `3 == 3` (true) |
| 논리 | `&& \|\| !` | `true && false` (false) |
| 조건 | `조건 ? A : B` | `age >= 20 ? "성인" : "미성년"` |

**꼭 알아야 할 점**

- **정수끼리 나누면 소수점이 버려진다.** `7 / 2`는 `3`이다. `3.5`를 원하면 `7 / 2.0` 또는 `(double)7 / 2`를 쓴다.
- **`%`(나머지) 연산자**는 짝수/홀수 판별(`n % 2 == 0`)에 자주 쓰인다.
- **`=`는 대입, `==`는 비교**이다.
- **`&&`는 둘 다 참, `||`는 하나라도 참**일 때 true이다. 앞쪽만으로 결과가 정해지면 뒤쪽은 계산하지 않는다(단락 평가).
- **`x++`와 `++x`**: 단독으로 쓰면 같다. 식 안에서는 `x++`는 연산 후 증가, `++x`는 증가 후 연산이다.
- **우선순위**: 곱셈·나눗셈이 덧셈·빼기보다 먼저이다. 헷갈리면 괄호를 친다.

```csharp
int a = 7, b = 2;
Console.WriteLine(a / b);          // 3
Console.WriteLine(a % b);          // 1
Console.WriteLine((double)a / b);  // 3.5
Console.WriteLine(a > b && b > 0); // True
```

**연습**: 초(예: 3725)를 입력받아 `1시간 2분 5초`처럼 출력한다. `/`와 `%`만 있으면 된다.

## 4장. 조건문과 반복문

코드는 기본적으로 위에서 아래로 한 줄씩 실행된다. 조건문은 상황에 따라 갈 길을 나누고, 반복문은 같은 일을 여러 번 하게 만든다.

**if / else if / else**

```csharp
int score = 85;
if (score >= 90)
    Console.WriteLine("A");
else if (score >= 80)
    Console.WriteLine("B");   // 여기가 실행됨
else
    Console.WriteLine("C 이하");
```

- 위에서부터 검사해서 **처음 참인 곳 하나만** 실행한다.
- 실행할 문장이 두 줄 이상이면 중괄호 `{ }`로 묶어야 한다. 한 줄이어도 묶는 습관을 들이면 실수가 줄어든다.

**switch**: 한 값을 여러 경우와 비교할 때 쓴다.

```csharp
int day = 3;
switch (day)
{
    case 1: Console.WriteLine("월"); break;
    case 2: Console.WriteLine("화"); break;
    case 3: Console.WriteLine("수"); break;
    default: Console.WriteLine("그 외"); break;
}

// switch 식: 값을 바로 돌려주는 짧은 형태
string name = day switch { 1 => "월", 2 => "화", 3 => "수", _ => "그 외" };
```

**반복문 네 가지**

| 반복문 | 언제 쓰나 |
| --- | --- |
| `for` | 반복 횟수가 정해져 있을 때 |
| `while` | 조건이 참인 동안 계속 (횟수를 모를 때) |
| `do-while` | 최소 한 번은 실행해야 할 때 |
| `foreach` | 배열이나 목록의 요소를 하나씩 꺼낼 때 |

```csharp
for (int i = 1; i <= 5; i++)       // 초기값; 조건; 증감
    Console.Write(i + " ");        // 1 2 3 4 5

int n = 10;
while (n > 0)
{
    n -= 3;                        // 조건이 언젠가 거짓이 되어야 함
}

int[] nums = { 3, 6, 9 };
foreach (int x in nums)
    Console.WriteLine(x);
```

- `break`: 반복문을 즉시 빠져나간다.
- `continue`: 이번 회차만 건너뛰고 다음 회차로 간다.
- **무한 루프 주의**: `while`의 조건이 계속 참이면 프로그램이 멈추지 않는다. 터미널에서 `Ctrl + C`로 멈출 수 있다.
- **중첩 반복문**: 반복문 안에 반복문을 넣을 수 있다. 구구단이 대표적인 예이다.

**연습**: (1) 1부터 100까지 짝수의 합을 구한다. (2) 구구단 2단부터 9단을 출력한다. (3) 1~100 사이 숫자를 맞히는 업다운 게임을 만든다(`Random.Shared.Next(1, 101)`).

## 5장. 배열과 문자열

배열은 같은 자료형의 값 여러 개를 한 이름으로 묶어 번호(인덱스)로 꺼내 쓰는 구조이다. 인덱스는 **0부터** 시작하고, 한 번 만든 배열은 크기를 바꿀 수 없다.

**배열**

```csharp
int[] scores = new int[3];        // 크기 3, 값은 모두 0
scores[0] = 90;
scores[1] = 80;
scores[2] = 70;

int[] nums = { 5, 3, 8 };          // 만들면서 바로 채우기
int[] more = [1, 2, 3];            // 최신 문법 (컬렉션 식)

Console.WriteLine(nums.Length);    // 3 (길이)
Console.WriteLine(nums[^1]);       // 8 (마지막 요소)

int sum = 0;
for (int i = 0; i < nums.Length; i++)   // < Length 주의 (<= 아님)
    sum += nums[i];
```

- 범위를 벗어난 인덱스(`nums[3]`)를 쓰면 `IndexOutOfRangeException` 오류가 난다. 초보자가 가장 자주 만나는 오류이다.
- `Array.Sort(nums)`로 정렬, `Array.Reverse(nums)`로 뒤집기를 할 수 있다.
- **2차원 배열**은 표처럼 행과 열이 있다. `int[,] grid = new int[3, 4];` 접근은 `grid[1, 2]`이다.
- 크기가 변하는 목록이 필요하면 `List<int>`를 쓴다(3부 컬렉션에서 자세히).

**문자열**

문자열(`string`)은 문자의 모음이다. 한 번 만들면 내용이 바뀌지 않아서(불변), 바꾸는 메서드는 항상 새 문자열을 돌려준다.

```csharp
string name = "홍길동";
int age = 20;

string a = "이름: " + name;                 // 연결
string b = $"{name}님은 {age}살입니다";     // 문자열 보간 (추천)
string c = $"평균: {85.456:F1}";            // 소수점 한 자리: 85.5

Console.WriteLine(name.Length);        // 3
Console.WriteLine(name[0]);            // 홍
Console.WriteLine(name.Contains("길")); // True
```

| 메서드 | 하는 일 |
| --- | --- |
| `ToUpper()` / `ToLower()` | 대문자 / 소문자로 |
| `Trim()` | 앞뒤 공백 제거 |
| `Substring(시작, 길이)` | 일부 잘라내기 |
| `Replace("a", "b")` | 바꾸기 |
| `Split(',')` | 구분자로 나눠 배열로 |
| `IndexOf("x")` | 위치 찾기 (없으면 -1) |
| `string.IsNullOrEmpty(s)` | 비었는지 검사 |

- 문자열 비교는 `==`로 내용을 비교한다.
- 반복문 안에서 문자열을 계속 이어 붙일 때는 `StringBuilder`가 빠르다. 불변이라 `+`를 할 때마다 새 문자열이 만들어지기 때문이다.
- `\n`은 줄바꿈, `\t`는 탭, `\"`는 큰따옴표이다(이스케이프 문자).

**연습**: (1) 점수 5개를 배열에 넣고 최댓값, 최솟값, 평균을 구한다. (2) 입력받은 문장을 거꾸로 출력한다. (3) 회문(기러기러기)인지 판별한다.

## 6장. 메서드

메서드는 특정 일을 하는 코드에 이름을 붙여 두고 필요할 때 불러 쓰는 묶음이다(다른 언어의 함수). 같은 코드를 여러 번 복사하고 있다면 메서드로 뽑아낼 때이다.

```csharp
// 반환형 이름(매개변수) { 본문 }
int Add(int a, int b)
{
    return a + b;          // 결과를 돌려줌
}

void SayHello(string name) // void = 돌려줄 값 없음
{
    Console.WriteLine($"안녕, {name}");
}

int result = Add(3, 4);    // 호출: result는 7
SayHello("철수");

// 본문이 한 줄이면 => 로 짧게
int Square(int x) => x * x;
```

**용어 정리**

- **매개변수(parameter)**: 메서드가 받는 변수(`a`, `b`)이다.
- **인수(argument)**: 호출할 때 실제로 넘기는 값(`3`, `4`)이다.
- **반환값**: `return`으로 돌려주는 값이다. `return`을 만나면 메서드는 즉시 끝난다.
- **지역 변수와 범위(스코프)**: 메서드 안에서 만든 변수는 그 중괄호 밖에서 보이지 않는다.

**값 전달과 ref / out**

기본적으로 인수는 **값이 복사**되어 전달된다. 메서드 안에서 바꿔도 원래 변수는 그대로이다.

```csharp
void Twice(int x) { x *= 2; }
void TwiceRef(ref int x) { x *= 2; }   // 원본을 직접 바꿈

int n = 5;
Twice(n);        // n은 그대로 5
TwiceRef(ref n); // n은 10

// out: 메서드가 값을 채워서 돌려줌 (int.TryParse가 이 방식)
void Divide(int a, int b, out int q, out int r) { q = a / b; r = a % b; }
Divide(17, 5, out int q, out int r);   // q=3, r=2
```

**편리한 기능**

- **오버로딩**: 이름은 같고 매개변수의 개수나 자료형이 다른 메서드를 여러 개 만들 수 있다. `Add(int, int)`와 `Add(double, double)`가 그 예다.
- **선택적 매개변수**: 기본값을 주면 생략할 수 있다. `void Greet(string name = "손님")`
- **명명된 인수**: 순서 대신 이름으로 넘길 수 있다. `Greet(name: "철수")`
- **재귀**: 메서드가 자기 자신을 부르는 것이다. 끝나는 조건이 꼭 있어야 한다.

```csharp
int Factorial(int n) => n <= 1 ? 1 : n * Factorial(n - 1);
```

**연습**: (1) 배열을 받아 최댓값을 돌려주는 `Max` 메서드를 만든다. (2) 소수인지 판별하는 `IsPrime(int n)`을 만들고 1~100의 소수를 출력한다. (3) 피보나치 수를 재귀와 반복문으로 각각 구해 본다.

## 7장. 예외 처리

예외는 프로그램이 실행되는 도중에 생기는 오류이다. 처리하지 않으면 프로그램이 그 자리에서 꺼지고, `try-catch`로 잡으면 오류 대신 다른 행동을 하게 만들 수 있다.

- **컴파일 오류**: 문법이 틀려 실행조차 안 된다. 편집기가 빨간 줄로 알려준다.
- **예외(런타임 오류)**: 문법은 맞지만 실행 중에 문제가 생긴다. 0으로 나누기, 숫자가 아닌 입력 등이 예이다.

```csharp
try
{
    Console.Write("나눌 수: ");
    int d = int.Parse(Console.ReadLine());
    Console.WriteLine(100 / d);
}
catch (FormatException)
{
    Console.WriteLine("숫자를 입력하세요");
}
catch (DivideByZeroException)
{
    Console.WriteLine("0으로 나눌 수 없습니다");
}
catch (Exception ex)              // 그 외 모든 예외 (맨 마지막에)
{
    Console.WriteLine($"오류: {ex.Message}");
}
finally
{
    Console.WriteLine("항상 실행");  // 예외가 나든 안 나든 실행
}
```

**자주 만나는 예외**

| 예외 | 언제 생기나 |
| --- | --- |
| `FormatException` | `int.Parse("abc")`처럼 형식이 맞지 않을 때 |
| `DivideByZeroException` | 정수를 0으로 나눌 때 |
| `IndexOutOfRangeException` | 배열 범위를 벗어난 인덱스 |
| `NullReferenceException` | 비어 있는(null) 변수를 사용할 때 |
| `OverflowException` | 값이 자료형 범위를 넘을 때 (checked 상황) |

**throw: 직접 예외 던지기**

```csharp
void SetAge(int age)
{
    if (age < 0)
        throw new ArgumentException("나이는 0 이상이어야 합니다");
}
```

**좋은 습관**

- 예측할 수 있는 문제는 예외보다 미리 검사하는 편이 낫다. 입력값은 `int.Parse` + `try-catch`보다 `int.TryParse`가 좋다.
- `catch`는 구체적인 예외부터 쓰고, `Exception`은 맨 마지막에 둔다.
- 예외를 잡고 아무것도 안 하는 빈 `catch { }`는 문제를 숨기므로 피한다.
- 오류 메시지(스택 트레이스)에는 어느 파일 몇 번째 줄에서 문제가 생겼는지 나온다. 꼭 읽는 습관을 들인다.

**연습**: 두 수와 연산자(+, -, \*, /)를 입력받는 계산기를 만들고, 잘못된 입력과 0으로 나누기에서 꺼지지 않게 한다.

## 1부 마무리

아래 미니 프로젝트 두 개를 힌트 없이 만들 수 있으면 1부는 끝난 것이다. 1~7장의 내용이 모두 들어간다.

- [ ] **성적 관리기**: 학생 수를 입력받고, 각 학생의 이름과 점수를 배열에 저장한다. 평균, 최고점 학생, 학점(A~F)을 출력한다. 잘못된 입력에도 꺼지지 않아야 한다.
- [ ] **메뉴형 콘솔 프로그램**: `1. 계산기  2. 업다운 게임  3. 구구단  0. 종료` 메뉴를 반복해서 보여 준다. 각 기능은 메서드 하나로 만든다.

**다음 단계: 2부 객체지향**

1부에서는 값과 메서드를 따로 다뤘다. 2부에서는 이 둘을 하나로 묶는 **클래스**를 배운다. 예를 들어 성적 관리기의 `이름 배열`과 `점수 배열`을 `Student`라는 클래스 하나로 바꾸게 된다. 이어서 다룰 주제는 클래스와 객체, 생성자, 속성, 캡슐화, 상속, 다형성, 인터페이스이다.

---

[← 목차로](README.md) · [다음: 2부 객체지향 →](02-oop.md)
