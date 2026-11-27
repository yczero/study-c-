# 3부. 중급

[← 2부 객체지향](02-oop.md) · [목차](README.md) · [4부 심화 →](04-advanced.md)

## 0. 3부 안내

3부는 실무 C# 코드에서 매일 보는 도구들을 배운다. 1·2부가 "문법"이었다면, 3부는 "데이터를 편하게 다루는 법"이다.

**공부 순서**

1. **15~16장 (제네릭, 컬렉션)**: 가장 먼저 본다. 이제부터 배열 대신 `List<T>`를 쓴다.
2. **17장 (람다)**: 18장 LINQ를 이해하려면 꼭 필요하다.
3. **18장 (LINQ)**: 3부의 하이라이트이다. 검색·정렬·집계를 한 줄로 한다.
4. **19~20장 (null, 패턴 매칭)**: 최신 C#다운 코드를 쓰는 데 필요하다.
5. **21~22장 (확장 메서드, 파일)**: 비교적 가벼우며, 22장을 배우면 프로그램을 꺼도 데이터가 남게 된다.

## 15장. 제네릭

제네릭은 **자료형을 나중에 정할 수 있게 비워 두는** 문법이다. 자료형을 `<T>`라는 자리표시로 적어 두면, 코드는 한 번만 쓰고 `int`, `string`, `Student` 등 어떤 타입에도 쓸 수 있다.

**왜 필요한가**

```csharp
// 제네릭 없이: 타입마다 같은 코드를 복사
void SwapInt(ref int a, ref int b) { (a, b) = (b, a); }
void SwapString(ref string a, ref string b) { (a, b) = (b, a); }

// 제네릭: 하나로 끝
void Swap<T>(ref T a, ref T b) { (a, b) = (b, a); }

int x = 1, y = 2;
Swap(ref x, ref y);          // T는 int로 자동 추론
string s1 = "A", s2 = "B";
Swap<string>(ref s1, ref s2); // 직접 지정해도 됨
```

`object`로 받아도 모든 타입을 담을 수는 있지만, 꺼낼 때마다 형변환이 필요하고 잘못된 타입이 들어가도 실행할 때까지 모른다. 제네릭은 컴파일할 때 타입을 검사하므로 안전하고, 박싱(12장)도 없어 빠르다.

**제네릭 클래스**

```csharp
var intBox = new Box<int>(42);
var strBox = new Box<string>("hi");
Console.WriteLine(intBox.Value + 1);   // 43, 형변환 필요 없음

class Box<T>
{
    public T Value { get; }
    public Box(T value) => Value = value;
}

class Pair<TKey, TValue>     // 타입 매개변수는 여러 개도 가능
{
    public TKey Key { get; set; } = default!;
    public TValue Value { get; set; } = default!;
}
```

**제약 조건 (where)**

`T`에 아무 타입이나 들어올 수 있으면, 컴파일러는 `T`가 무엇을 할 수 있는지 모른다. `where`로 조건을 걸면 그 기능을 쓸 수 있다.

```csharp
T Max<T>(T a, T b) where T : IComparable<T>   // 비교 가능한 타입만
    => a.CompareTo(b) >= 0 ? a : b;

Console.WriteLine(Max(3, 7));          // 7
Console.WriteLine(Max("apple", "kiwi")); // kiwi
```

| 제약 | 의미 |
| --- | --- |
| `where T : class` | 참조 타입만 |
| `where T : struct` | 값 타입만 |
| `where T : new()` | 매개변수 없는 생성자가 있는 타입 (`new T()` 가능) |
| `where T : Animal` | Animal이거나 그 자식 |
| `where T : IComparable<T>` | 그 인터페이스를 구현한 타입 |
| `where T : notnull` | null이 아닌 타입 |

- `T`는 관례적인 이름일 뿐이다. 두 개 이상이면 `TKey`, `TValue`처럼 의미를 붙인다.
- `default(T)` 또는 `default`는 그 타입의 기본값(숫자는 0, 참조 타입은 null)이다.
- 직접 제네릭을 만들 일보다 **쓸 일이 훨씬 많다.** 다음 장의 컬렉션이 모두 제네릭이다.

**연습**: 마지막에 넣은 것이 먼저 나오는 `MyStack<T>` 클래스를 배열로 만든다(`Push`, `Pop`, `Count`). `int`와 `string`으로 각각 테스트한다.

## 16장. 컬렉션

컬렉션은 여러 값을 담는 그릇이고, 배열과 달리 **크기가 자동으로 늘어난다.** 모두 `System.Collections.Generic`에 있고 기본으로 불러와 있어서 바로 쓸 수 있다. 상황에 맞는 그릇을 고르는 것이 핵심이다.

| 컬렉션 | 특징 | 언제 쓰나 |
| --- | --- | --- |
| `List<T>` | 순서 있는 목록, 인덱스로 접근 | 가장 기본. 고민되면 이것 |
| `Dictionary<TKey, TValue>` | 키로 값을 빠르게 찾음 (키 중복 불가) | 학번→학생, 단어→뜻 |
| `HashSet<T>` | 중복 없는 모음, 포함 여부 확인이 빠름 | 방문한 페이지, 중복 제거 |
| `Queue<T>` | 먼저 들어간 게 먼저 나옴 (FIFO) | 대기열, 주문 처리 |
| `Stack<T>` | 마지막에 들어간 게 먼저 나옴 (LIFO) | 되돌리기, 괄호 검사 |

**List&lt;T&gt;**

```csharp
List<string> names = ["철수", "영희"];   // new List<string>() 도 가능
names.Add("민수");             // 끝에 추가
names.Insert(0, "지민");        // 위치 지정 추가
names.Remove("영희");          // 값으로 삭제
names.RemoveAt(0);              // 인덱스로 삭제

Console.WriteLine(names.Count);           // 개수 (배열은 Length)
Console.WriteLine(names[0]);
Console.WriteLine(names.Contains("민수")); // True
names.Sort();

foreach (var n in names) Console.WriteLine(n);
```

**Dictionary&lt;TKey, TValue&gt;**

```csharp
Dictionary<string, int> stock = new()
{
    ["사과"] = 10,
    ["바나나"] = 5,
};
stock["포도"] = 7;          // 없으면 추가, 있으면 덮어쓰기
stock["사과"] -= 1;

// Console.WriteLine(stock["딸기"]);  ← 없는 키: KeyNotFoundException
if (stock.TryGetValue("딸기", out int count))   // 안전하게 꺼내기
    Console.WriteLine(count);

foreach (var (fruit, qty) in stock)
    Console.WriteLine($"{fruit}: {qty}개");
```

**HashSet, Queue, Stack**

```csharp
var seen = new HashSet<int>();
seen.Add(3); seen.Add(3);            // 두 번째는 무시됨 (false 반환)
Console.WriteLine(seen.Count);       // 1

var line = new Queue<string>();
line.Enqueue("손님1"); line.Enqueue("손님2");
Console.WriteLine(line.Dequeue());   // 손님1

var undo = new Stack<string>();
undo.Push("입력A"); undo.Push("입력B");
Console.WriteLine(undo.Pop());       // 입력B
```

**주의할 점**

- `foreach`로 돌면서 그 컬렉션에 추가·삭제하면 `InvalidOperationException`이 난다. 삭제는 `RemoveAll(조건)`이나 뒤에서부터 도는 `for`를 쓴다.
- `List`에서 `Contains`는 처음부터 끝까지 하나씩 확인한다. 데이터가 많고 검색이 잦으면 `HashSet`이나 `Dictionary`가 훨씬 빠르다.
- `Dictionary`는 순서를 보장하지 않는다고 생각한다. 정렬이 필요하면 `SortedDictionary`를 쓴다.

**연습**: (1) 문장을 입력받아 단어별 등장 횟수를 `Dictionary`로 센다. (2) 문자열의 괄호 `()[]{}`가 올바르게 짝지어졌는지 `Stack`으로 검사한다.

## 17장. 델리게이트, 람다, 이벤트

델리게이트는 **메서드를 변수처럼 담아서 전달할 수 있게** 하는 타입이고, 람다는 그 메서드를 이름 없이 짧게 쓰는 방법이다. "어떻게 처리할지"를 호출하는 쪽이 정해서 넘겨줄 수 있다는 것이 핵심이다.

**람다식**

```csharp
// (매개변수) => 식
Func<int, int> square = x => x * x;
Func<int, int, int> add = (a, b) => a + b;
Action<string> greet = name => Console.WriteLine($"안녕, {name}");
Predicate<int> isEven = n => n % 2 == 0;

Console.WriteLine(square(5));   // 25
greet("철수");

// 본문이 길면 중괄호
Func<int, string> grade = score =>
{
    if (score >= 90) return "A";
    return "B 이하";
};
```

**미리 만들어진 델리게이트 타입**: 직접 선언할 일은 드물고, 거의 항상 이 세 가지를 쓴다.

| 타입 | 의미 | 예 |
| --- | --- | --- |
| `Action<T...>` | 반환값 없음 (void) | `Action<string>` |
| `Func<T..., TResult>` | 마지막 타입이 반환형 | `Func<int, int, int>` = int 두 개 받아 int 반환 |
| `Predicate<T>` | bool 반환 (조건 검사) | `Func<T, bool>`과 같은 역할 |

**왜 쓰는가: 동작을 넘겨주기**

```csharp
List<int> nums = [5, 12, 8, 21, 3];

var big = nums.FindAll(n => n > 7);        // [12, 8, 21]
nums.RemoveAll(n => n % 2 == 1);           // 홀수 삭제
nums.Sort((a, b) => b.CompareTo(a));        // 내림차순

// 내가 만든 메서드도 동작을 받을 수 있음
void Repeat(int times, Action<int> action)
{
    for (int i = 0; i < times; i++) action(i);
}
Repeat(3, i => Console.WriteLine($"{i}번째"));
```

"무엇을 거를지"만 람다로 넘기고, 반복문은 `FindAll`이 대신 돌려준다. 이 방식이 다음 장 LINQ의 바탕이다.

**이벤트**

이벤트는 "무슨 일이 생기면 알려 달라"고 구독하는 구조이다. 버튼 클릭, 다운로드 완료처럼 UI와 게임(Unity)에서 많이 쓴다.

```csharp
var alarm = new Alarm();
alarm.Rang += msg => Console.WriteLine($"[알림] {msg}");   // 구독 (+=)
alarm.Rang += msg => Console.WriteLine("일어나!");         // 여러 개 가능
alarm.Ring();

class Alarm
{
    public event Action<string>? Rang;     // 이벤트 선언
    public void Ring() => Rang?.Invoke("7시입니다");  // 구독자 모두 호출
}
```

- 구독은 `+=`, 취소는 `-=`이다.
- `event`를 붙이면 밖에서는 구독만 할 수 있고, 실행(`Invoke`)은 클래스 안에서만 할 수 있다.
- `?.Invoke`는 구독자가 없을 때(null) 오류가 나지 않게 한다(19장).

**연습**: `List<Student>`를 점수 내림차순으로 `Sort`하고, 80점 이상만 `FindAll`로 골라 출력한다. 둘 다 람다로 작성한다.

## 18장. LINQ

LINQ는 컬렉션에서 **골라내고, 바꾸고, 정렬하고, 집계하는 일을 메서드 연결로 한 줄에** 하는 기능이다. 반복문과 if로 10줄 걸리던 코드가 짧고 읽기 쉬워진다. 배열, List, Dictionary 등 `foreach`로 돌 수 있는 모든 것에 쓸 수 있다.

```csharp
List<Student> students =
[
    new("철수", "컴공", 85),
    new("영희", "경영", 92),
    new("민수", "컴공", 71),
    new("지수", "컴공", 96),
];

// 컴공 학생 중 80점 이상을 점수 높은 순으로, 이름만
var names = students
    .Where(s => s.Major == "컴공" && s.Score >= 80)   // 골라내기
    .OrderByDescending(s => s.Score)                    // 정렬
    .Select(s => s.Name)                                // 바꾸기
    .ToList();                                          // 결과를 List로
// ["지수", "철수"]

record Student(string Name, string Major, int Score);
```

**자주 쓰는 메서드**

| 분류 | 메서드 | 하는 일 |
| --- | --- | --- |
| 골라내기 | `Where(조건)` | 조건에 맞는 것만 |
| 바꾸기 | `Select(변환)` | 각 요소를 다른 값으로 |
| 정렬 | `OrderBy`, `OrderByDescending`, `ThenBy` | 오름/내림차순, 2차 기준 |
| 집계 | `Count`, `Sum`, `Average`, `Min`, `Max` | 개수, 합계, 평균 등 |
| 하나 꺼내기 | `First`, `FirstOrDefault`, `Single` | 첫 번째 요소 (없으면 예외 / 기본값) |
| 검사 | `Any`, `All`, `Contains` | 하나라도 / 모두 / 포함 여부 |
| 묶기 | `GroupBy(키)` | 키별로 그룹 |
| 자르기 | `Take(n)`, `Skip(n)`, `Distinct()` | 앞 n개 / n개 건너뛰기 / 중복 제거 |
| 변환 | `ToList`, `ToArray`, `ToDictionary` | 결과를 컬렉션으로 |

```csharp
double avg = students.Average(s => s.Score);                  // 86
bool anyFail = students.Any(s => s.Score < 60);              // False
var top = students.MaxBy(s => s.Score);                      // 지수
var first = students.FirstOrDefault(s => s.Major == "미술");  // null

foreach (var g in students.GroupBy(s => s.Major))
    Console.WriteLine($"{g.Key}: {g.Count()}명, 평균 {g.Average(s => s.Score):F1}");
// 컴공: 3명, 평균 84.0 / 경영: 1명, 평균 92.0
```

**꼭 알아야 할 점**

- **지연 실행**: `Where`, `Select` 등은 바로 실행되지 않고, `foreach`나 `ToList()`를 만나야 실행된다. 같은 쿼리를 여러 번 쓰면 매번 다시 계산하므로, 한 번 `ToList()`로 결과를 만들어 두면 좋다.
- **`First` vs `FirstOrDefault`**: 없을 수도 있으면 `FirstOrDefault`를 쓰고 null인지 확인한다.
- LINQ는 원본을 바꾸지 않고 항상 새 결과를 만든다.
- SQL과 비슷한 **쿼리 구문**(`from s in students where s.Score > 80 select s.Name`)도 있지만, 실무에서는 위의 **메서드 구문**을 더 많이 쓴다.

**연습**: 1~100 중 3의 배수의 제곱을 합친 값을 LINQ 한 줄로 구한다(`Enumerable.Range(1, 100)`). 그다음 위 `students`를 학과별 최고점 학생 이름으로 출력한다.

## 19장. null 다루기

`null`은 "가리키는 객체가 없음"이고, null인 변수의 멤버를 쓰면 `NullReferenceException`이 난다. 최신 C#은 **어떤 변수가 null이 될 수 있는지를 타입에 표시**하고, 위험한 곳을 컴파일러가 경고로 알려 준다.

**널 허용 타입 (`?`)**

```csharp
string name = "철수";     // null이 없는 문자열
string? nick = null;       // null일 수 있음을 표시

int score = 0;             // 값 타입은 원래 null 불가
int? bonus = null;         // int?는 null 가능

// Console.WriteLine(nick.Length);  ← 경고: nick은 null일 수 있음
if (nick != null)
    Console.WriteLine(nick.Length);  // 확인한 뒤에는 경고 없음
```

- 새 프로젝트는 이 기능(nullable)이 켜져 있다. 경고는 실행을 막지 않지만 무시하지 말고 하나씩 해결하는 습관을 들인다.
- `Console.ReadLine()`의 반환형이 `string?`인 이유도 입력이 끝나면 null이 올 수 있기 때문이다.

**null 관련 연산자**

| 연산자 | 이름 | 의미 |
| --- | --- | --- |
| `a?.B` | null 조건부 연산자 | a가 null이면 멈추고 null, 아니면 a.B |
| `a ?? b` | null 병합 연산자 | a가 null이면 b |
| `a ??= b` | null 병합 대입 | a가 null일 때만 b를 대입 |
| `a!` | null 용서 연산자 | "null 아니니 믿어"라고 경고를 끔 (검사는 안 함) |

```csharp
string? input = Console.ReadLine();

int len = input?.Length ?? 0;                // null이면 0
string display = input ?? "(입력 없음)";

List<string>? cache = null;
cache ??= new List<string>();                 // 처음 쓸 때만 생성

Student? s = FindStudent("철수");
Console.WriteLine(s?.Name ?? "없는 학생");
```

**원칙**

- 반환값이나 속성이 "없을 수도 있다"면 `?`를 붙여 솔직하게 표시한다.
- `!`는 경고만 숨길 뿐이라, 틀리면 여전히 예외가 난다. 확실할 때만 쓴다.
- 컬렉션은 null 대신 빈 컬렉션(`[]`)을 돌려주면 호출하는 쪽이 편하다.
- null 확인은 `x is null`, `x is not null`로도 쓸 수 있다(20장 패턴 매칭).

**연습**: 이름으로 학생을 찾아 `Student?`를 돌려주는 메서드를 만들고, 결과를 `?.`와 `??`만으로 안전하게 출력한다.

## 20장. 패턴 매칭

패턴 매칭은 값이 **어떤 모양(타입, 범위, 속성 값)인지 검사하고, 맞으면 그 자리에서 꺼내 쓰는** 문법이다. 길게 이어지는 if-else를 짧고 읽기 쉽게 바꿀 수 있다. `is`와 `switch` 식에서 쓴다.

**타입 패턴 (is)**

```csharp
object value = 42;

if (value is int n)              // int이면 n에 담아서
    Console.WriteLine(n + 1);    // 바로 사용 (형변환 필요 없음)

if (value is not null) { /* ... */ }
```

**switch 식과 여러 패턴**

```csharp
string Grade(int score) => score switch
{
    >= 90 => "A",                  // 관계 패턴
    >= 80 and < 90 => "B",         // and / or / not 조합
    >= 0 => "C 이하",
    _ => "잘못된 점수"              // _ = 나머지 전부
};

double Area(Shape s) => s switch
{
    Circle c => Math.PI * c.R * c.R,         // 타입 패턴
    Rect { W: var w, H: var h } => w * h,    // 속성 패턴
    _ => 0
};

string Discount(Customer c) => c switch
{
    { IsVip: true } => "20%",
    { Age: < 20 or >= 65 } => "10%",
    _ => "없음"
};
```

**패턴 종류 정리**

| 패턴 | 모양 | 검사하는 것 |
| --- | --- | --- |
| 상수 | `3`, `"월"`, `null` | 값이 같은가 |
| 타입 | `Circle c` | 그 타입인가 (맞으면 변수에 담김) |
| 관계 | `>= 90`, `< 0` | 범위에 드는가 |
| 논리 | `and`, `or`, `not` | 패턴 조합 |
| 속성 | `{ Age: > 19 }` | 속성 값이 조건에 맞는가 |
| 리스트 | `[1, .., 9]` | 배열/리스트의 모양 |
| 버림 | `_` | 무엇이든 (기본값) |

```csharp
int[] arr = [1, 5, 9];
if (arr is [1, .., 9]) Console.WriteLine("1로 시작해서 9로 끝남");
if (arr is [var first, ..]) Console.WriteLine($"첫 값: {first}");
```

- switch 식은 **위에서부터 검사**해서 처음 맞는 것을 고른다. 넓은 조건을 위에 두면 아래는 영원히 실행되지 않고, 컴파일러가 오류로 알려 준다.
- 모든 경우를 다루지 않으면 경고가 뜨므로 마지막에 `_`를 둔다.
- C# 15의 공용체 타입(union)은 "이 값은 A, B, C 중 하나"를 타입으로 정해 두고 패턴 매칭으로 빠짐없이 처리하는 기능이다. 5부에서 다룬다.

**연습**: 13장의 `OrderStatus`를 받아 안내 문구를 돌려주는 메서드를 switch 식으로 만들고, 나이와 학생 여부로 영화 요금을 정하는 메서드를 속성 패턴으로 만든다.

## 21장. 확장 메서드와 확장 멤버

확장 메서드는 **내가 고칠 수 없는 클래스(예: `string`, `int`)에 메서드를 추가한 것처럼** 쓰게 해 준다. 실제로는 static 메서드인데, 호출할 때는 원래 있던 메서드처럼 점(`.`)으로 부른다. 18장의 LINQ(`Where`, `Select`)가 모두 확장 메서드이다.

**전통적인 방식 (this 매개변수)**

```csharp
Console.WriteLine("hello".Shout());     // HELLO!
Console.WriteLine(7.IsEven());          // False

static class MyExtensions               // static 클래스 안에
{
    // 첫 번째 매개변수 앞에 this = 이 타입을 확장
    public static string Shout(this string s) => s.ToUpper() + "!";
    public static bool IsEven(this int n) => n % 2 == 0;
}
```

**최신 방식 (C# 14의 extension 블록)**

C# 14부터는 메서드뿐 아니라 **속성**도 확장할 수 있는 `extension` 블록이 생겼고, C# 15에서는 인덱서까지 확장할 수 있게 됐다.

```csharp
Console.WriteLine("   ".IsBlank);     // True  (속성처럼 괄호 없이)
Console.WriteLine("hi".Shout());     // HI!

static class StringExtensions
{
    extension(string s)               // string을 확장하는 블록
    {
        public bool IsBlank => string.IsNullOrWhiteSpace(s);   // 확장 속성
        public string Shout() => s.ToUpper() + "!";           // 확장 메서드
    }
}
```

**알아두면 좋은 점**

- 확장은 원래 클래스를 바꾸지 않는다. 그래서 `private` 멤버에는 접근할 수 없고, 공개된 멤버만 쓴다.
- 클래스에 같은 이름의 메서드가 이미 있으면 원래 메서드가 우선이다.
- 다른 네임스페이스에 만들었다면 `using`으로 불러와야 보인다. LINQ가 `using System.Linq;`를 필요로 하는 이유이다(최신 프로젝트는 자동).
- 남용하면 "이 메서드가 어디서 왔지?"가 헷갈린다. 자주 쓰는 작은 도구에만 쓴다.
- 교재와 기존 코드는 대부분 `this` 방식이니 두 방식을 모두 읽을 줄 알아야 한다.

**연습**: `int`에 `IsPrime()` 확장 메서드를, `string`에 단어 수를 돌려주는 `WordCount` 확장 속성을 만들고 `"hello world".WordCount`로 테스트한다.

## 22장. 파일 입출력과 JSON

지금까지 만든 프로그램은 꺼지면 데이터가 사라졌다. 파일에 저장하면 다음에 다시 불러올 수 있고, **객체를 JSON 텍스트로 바꾸어(직렬화) 저장하면 클래스 구조 그대로** 복원할 수 있다.

**텍스트 파일: File 클래스**

```csharp
string path = "memo.txt";

File.WriteAllText(path, "첫 줄\n");            // 새로 쓰기 (있으면 덮어씀)
File.AppendAllText(path, "추가한 줄\n");     // 뒤에 덧붙이기
File.WriteAllLines(path, ["사과", "바나나"]); // 줄 단위로 쓰기

if (File.Exists(path))
{
    string all = File.ReadAllText(path);           // 통째로 읽기
    string[] lines = File.ReadAllLines(path);      // 줄 단위로 읽기
}
```

- 경로만 쓰면 실행 폴더(보통 `bin/Debug/...`) 기준이다. 폴더를 합칠 때는 `Path.Combine("data", "memo.txt")`를 쓰면 운영체제별 구분자를 알아서 맞춰 준다.
- 파일이 없거나 권한이 없으면 예외가 난다(`FileNotFoundException`, `UnauthorizedAccessException`). 7장의 `try-catch`로 감싼다.

**큰 파일: StreamReader와 using**

```csharp
using (var reader = new StreamReader("big.txt"))   // 끝나면 자동으로 파일을 닫음
{
    string? line;
    while ((line = reader.ReadLine()) != null)
        Console.WriteLine(line);
}

using var writer = new StreamWriter("log.txt");  // 줄임 형태: 블록이 끝나면 닫힘
writer.WriteLine("기록");
```

파일은 다 쓴 뒤 반드시 닫아야 한다. `using`은 예외가 나도 알아서 닫아 주므로 항상 쓴다(`IDisposable`을 구현한 객체에 쓸 수 있음).

**JSON 직렬화: System.Text.Json**

```csharp
using System.Text.Encodings.Web;
using System.Text.Json;
using System.Text.Unicode;

List<Student> students = [new("철수", 85), new("영희", 92)];
var options = new JsonSerializerOptions
{
    WriteIndented = true,                                   // 보기 좋게 줄바꿈
    Encoder = JavaScriptEncoder.Create(UnicodeRanges.All),  // 한글을 그대로 저장
};

// 객체 → JSON (직렬화) → 파일
string json = JsonSerializer.Serialize(students, options);
File.WriteAllText("students.json", json);

// 파일 → JSON → 객체 (역직렬화)
string loaded = File.ReadAllText("students.json");
var back = JsonSerializer.Deserialize<List<Student>>(loaded) ?? [];
Console.WriteLine(back[1].Name);   // 영희

record Student(string Name, int Score);
```

저장된 `students.json` 파일의 모습은 이렇다.

```json
[
  {
    "Name": "철수",
    "Score": 85
  },
  {
    "Name": "영희",
    "Score": 92
  }
]
```

`Encoder` 설정을 빼면 한글이 글자 대신 `철수` 같은 유니코드 번호로 저장된다. 다시 읽으면 올바르게 복원되지만, 파일을 사람이 열어 보기는 어렵다.

- 기본적으로 **public 속성**만 저장된다. private 필드는 빠진다.
- `Deserialize`는 파일 내용이 비어 있으면 null을 돌려줄 수 있어서 `?? []`로 대비한다(19장).
- JSON은 웹 API가 데이터를 주고받는 표준 형식이라, 나중에 서버나 앱을 만들 때도 그대로 쓰인다.

**연습**: 16장의 단어별 등장 횟수 `Dictionary`를 JSON 파일로 저장하고, 프로그램을 다시 실행했을 때 불러와서 이어 세게 한다.

## 3부 마무리

아래 프로젝트 하나에 3부의 거의 모든 내용이 들어간다. 2부에서 만든 도서관 프로그램을 고쳐 써도 된다.

- [ ] **가계부 프로그램**: `record Entry(DateOnly Date, string Category, decimal Amount)`로 수입·지출을 기록한다. `List<Entry>`에 담고(16장), 카테고리별 합계와 이번 달 지출 상위 3개를 LINQ로 출력한다(18장). 금액 구간별 표시는 switch 식(20장)으로, 저장과 불러오기는 JSON 파일(22장)로 한다. 파일이 없는 첫 실행에도 꺼지지 않아야 한다(19장).

**다음 단계: 4부 심화**

지금까지의 코드는 한 번에 한 가지 일만 했다. 4부에서는 다운로드를 기다리는 동안 화면이 멈추지 않게 하는 비동기(async/await), 여러 일을 동시에 하는 멀티스레딩, 메모리 관리와 성능, 의존성 주입 같은 설계 패턴을 배운다.

---

[← 2부 객체지향](02-oop.md) · [목차](README.md) · [4부 심화 →](04-advanced.md)
