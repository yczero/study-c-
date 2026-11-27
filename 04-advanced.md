# 4부. 심화

[← 3부 중급](03-intermediate.md) · [목차](README.md) · [5부 최신 기능 + 부록 →](05-latest-and-appendix.md)

## 0. 4부 안내

4부는 "돌아가는 코드"를 "실무에서 쓸 수 있는 코드"로 만드는 내용이다. 1학년이 한 번에 다 이해하기는 어려우니, **23장(비동기)과 30장(의존성 주입)을 먼저 확실히** 보고 나머지는 개념만 잡아 두어도 충분하다.

| 장 | 중요도 | 이유 |
| --- | --- | --- |
| 23장 비동기 | 필수 | 웹, 앱, 게임 어디서나 매일 쓴다 |
| 30장 의존성 주입 | 필수 | ASP.NET Core 같은 프레임워크의 기본 구조이다 |
| 24장 멀티스레딩 | 중요 | 동시성 버그를 이해하려면 필요하다 |
| 25장 메모리 관리 | 중요 | `using`과 `IDisposable`은 꼭 알아야 한다 |
| 26~29장 | 개념만 | 성능, 라이브러리 개발, 특수한 상황에서 쓴다 |

## 23장. 비동기 프로그래밍

비동기는 **오래 걸리는 작업(파일, 네트워크, DB)을 기다리는 동안 다른 일을 할 수 있게** 하는 방식이다. 라면 물이 끓는 동안 가만히 서 있지 않고 재료를 써는 것과 같다. C#에서는 `async`와 `await` 두 키워드로 비동기 코드를 동기 코드처럼 읽기 쉽게 쓴다.

```csharp
using var http = new HttpClient();

Console.WriteLine("다운로드 시작");
string html = await http.GetStringAsync("https://example.com");  // 기다리는 동안 스레드를 붙잡지 않음
Console.WriteLine($"글자 수: {html.Length}");

int total = await SumAsync(10);
Console.WriteLine(total);

async Task<int> SumAsync(int n)      // async 메서드는 Task 또는 Task<T>를 반환
{
    await Task.Delay(1000);          // 1초 기다림 (Thread.Sleep과 달리 막지 않음)
    return n * (n + 1) / 2;
}
```

**핵심 개념**

| 용어 | 의미 |
| --- | --- |
| `Task` | "언젠가 끝날 작업"을 나타내는 객체. 반환값이 없으면 `Task`, 있으면 `Task<T>` |
| `async` | 이 메서드 안에서 `await`를 쓰겠다는 표시 |
| `await` | 작업이 끝날 때까지 기다리되, 그동안 스레드는 다른 일을 하러 감 |
| `Task.WhenAll` | 여러 작업을 동시에 시작하고 모두 끝날 때까지 기다림 |
| `CancellationToken` | 진행 중인 작업에 "그만해"라고 알리는 신호 |

**여러 작업을 동시에**

```csharp
// 순서대로: 1초 + 1초 + 1초 = 약 3초
await Task.Delay(1000); await Task.Delay(1000); await Task.Delay(1000);

// 동시에: 약 1초
await Task.WhenAll(Task.Delay(1000), Task.Delay(1000), Task.Delay(1000));
```

**취소하기**

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));  // 3초 뒤 자동 취소
try
{
    await LongWorkAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("시간 초과로 취소됨");
}

async Task LongWorkAsync(CancellationToken token)
{
    for (int i = 0; i < 10; i++)
    {
        token.ThrowIfCancellationRequested();   // 취소 요청이 왔으면 멈춤
        await Task.Delay(1000, token);
    }
}
```

**규칙과 흔한 실수**

- 비동기 메서드 이름은 `~Async`로 끝내는 것이 관례이다.
- `async void`는 쓰지 않는다. 예외를 잡을 수 없기 때문이다. 유일한 예외는 버튼 클릭 같은 이벤트 처리기이다.
- `.Result`나 `.Wait()`로 억지로 기다리면 프로그램이 멈출(교착 상태) 수 있다. 끝까지 `await`를 쓴다("async all the way").
- 비동기는 "빨라지는 것"이 아니라 "기다리는 동안 막히지 않는 것"이다. 계산이 무거운 작업은 24장의 병렬 처리가 맞다.
- .NET 11부터는 런타임이 비동기를 직접 처리하는 방식(Runtime Async)이 도입돼 내부 성능이 좋아졌지만, 코드 작성법은 그대로이다.

**연습**: URL 세 개를 `Task.WhenAll`로 동시에 내려받아 각 페이지의 글자 수를 출력하고, 순서대로 받았을 때와 시간을 비교한다(`Stopwatch` 사용).

## 24장. 멀티스레딩과 동기화

스레드는 프로그램 안에서 코드를 실행하는 하나의 흐름이고, 멀티스레딩은 **여러 흐름이 동시에 실행되는 것**이다. CPU 코어가 여러 개라서 무거운 계산을 나눠 처리하면 빨라진다. 대신 여러 스레드가 같은 데이터를 동시에 건드리면 결과가 꼬이는 문제가 생기고, 이를 막는 것이 동기화이다.

**병렬 처리: Parallel**

```csharp
// 1부터 1000만까지 소수 개수 세기 (IsPrime은 6장 연습에서 만든 메서드)
int count = 0;
Parallel.For(1, 10_000_001, n =>
{
    if (IsPrime(n))
        Interlocked.Increment(ref count);   // 안전하게 1 더하기
});
Console.WriteLine(count);
```

- `Parallel.For`, `Parallel.ForEach`는 반복을 여러 코어에 나눠 실행한다.
- 계산이 무거운 작업(CPU 작업)에 쓴다. 네트워크나 파일 대기(I/O 작업)에는 23장의 async/await가 맞다.
- `Task.Run(() => ...)`으로 작업 하나를 백그라운드 스레드에 보낼 수도 있다.

**경쟁 상태(Race Condition)**

```csharp
int counter = 0;
Parallel.For(0, 100_000, _ => counter++);
Console.WriteLine(counter);   // 100000이 아니라 매번 다른 작은 값
```

`counter++`는 "읽기 → 1 더하기 → 쓰기" 세 단계이다. 두 스레드가 같은 값을 동시에 읽으면 하나의 증가가 사라진다. 이런 버그는 가끔만 나타나서 찾기 매우 어렵다.

**해결 방법**

| 방법 | 쓰는 곳 |
| --- | --- |
| `lock` | 한 번에 한 스레드만 들어가게 막는 구역. 가장 기본 |
| `Interlocked` | 숫자 증가, 교환 같은 단순 연산을 안전하게 |
| `ConcurrentDictionary`, `ConcurrentQueue` | 여러 스레드가 함께 써도 안전한 컬렉션 |
| `Channel<T>` | 생산자가 넣고 소비자가 꺼내는 안전한 통로 |

```csharp
var gate = new Lock();     // .NET 9부터 생긴 전용 잠금 타입 (예전 코드는 new object())
int total = 0;
Parallel.For(0, 100_000, _ =>
{
    lock (gate)            // 이 블록은 한 번에 한 스레드만 실행
    {
        total++;
    }
});
Console.WriteLine(total);  // 100000
```

**Channel: 생산자와 소비자**

```csharp
using System.Threading.Channels;

var channel = Channel.CreateUnbounded<int>();

var producer = Task.Run(async () =>
{
    for (int i = 1; i <= 5; i++)
        await channel.Writer.WriteAsync(i);
    channel.Writer.Complete();               // 더 이상 넣을 것 없음
});

await foreach (var item in channel.Reader.ReadAllAsync())
    Console.WriteLine($"처리: {item}");

await producer;
```

**주의할 점**

- `lock` 안에서는 짧은 작업만 한다. 안에서 `await`는 쓸 수 없다.
- 두 스레드가 서로 상대의 잠금을 기다리면 영원히 멈춘다(교착 상태, Deadlock). 잠금은 항상 같은 순서로 잡는다.
- 가장 좋은 동기화는 공유하지 않는 것이다. 각 스레드가 자기 데이터만 다루고 마지막에 합치는 구조를 먼저 고민한다.

**연습**: 경쟁 상태 예제를 직접 실행해 결과가 매번 다른지 확인하고, `lock`과 `Interlocked` 두 방법으로 각각 고쳐 본다.

## 25장. 메모리 관리

C#은 **가비지 컬렉터(GC)가 더 이상 쓰지 않는 객체의 메모리를 자동으로 정리**한다. C처럼 직접 `free`할 필요가 없다. 다만 GC는 메모리만 치울 뿐이라, 파일·네트워크 연결·DB 연결 같은 외부 자원은 우리가 직접 돌려줘야 한다.

**GC가 하는 일**

- 힙(12장)에 만든 객체 중 어느 변수에서도 가리키지 않는 것을 찾아 메모리를 회수한다.
- 객체를 세대(0, 1, 2세대)로 나눈다. 대부분의 객체는 만들어지자마자 금방 버려지므로, 젊은 세대(0세대)를 자주 치우고 오래 살아남은 객체는 가끔 검사한다.
- 언제 실행될지는 런타임이 정한다. `GC.Collect()`로 직접 부르는 것은 거의 항상 필요 없다.

**IDisposable과 using**

외부 자원을 쓰는 클래스는 `IDisposable`을 구현하고 `Dispose()`에서 자원을 돌려준다. 쓰는 쪽은 `using`으로 감싸면 예외가 나도 반드시 `Dispose()`가 불린다.

```csharp
using (var log = new LogWriter("app.log"))
{
    log.Write("시작");
}                                 // 여기서 log.Dispose() 자동 호출

using var log2 = new LogWriter("app2.log");   // 줄임 형태: 메서드가 끝날 때 Dispose

class LogWriter : IDisposable
{
    private readonly StreamWriter writer;
    public LogWriter(string path) => writer = new StreamWriter(path);
    public void Write(string msg) => writer.WriteLine(msg);
    public void Dispose() => writer.Dispose();   // 내가 가진 자원도 정리
}
```

- 비동기 자원은 `IAsyncDisposable`과 `await using`을 쓴다.
- `IDisposable`을 구현한 객체를 만들었다면 `using`을 붙이는 것이 기본 습관이다. 편집기도 빠뜨리면 경고를 보여 준다.

**메모리 누수는 C#에서도 생긴다**

GC는 "아무도 가리키지 않는" 객체만 치운다. 누군가 계속 가리키고 있으면 안 쓰더라도 남아 있다.

- static 리스트에 계속 추가만 하고 지우지 않는 경우
- 이벤트를 `+=`로 구독하고 `-=`로 해제하지 않은 경우 (17장)
- 캐시에 크기 제한이 없는 경우

**연습**: 생성될 때 "열림", `Dispose`될 때 "닫힘"을 출력하는 클래스를 만들고, `using` 블록 안에서 예외를 던져도 "닫힘"이 출력되는지 확인한다.

## 26장. 고성능 C#: Span과 도구들

이 장의 도구들은 모두 **불필요한 복사와 힙 할당을 줄여서 GC의 부담을 덜어주는** 것이 목적이다. 일반적인 앱 코드에서는 드물게 쓰지만, .NET 내부와 게임, 서버 라이브러리에서 많이 쓰이므로 "무엇을 하는 도구인지"는 알아둔다.

**Span&lt;T&gt;: 복사 없는 창문**

`Span<T>`는 배열이나 문자열의 일부분을 **복사하지 않고 가리키는 창문**이다. `Substring`은 새 문자열을 만들지만, `Span`은 원본 위에 어디서부터 어디까지라는 표시만 둔다.

```csharp
int[] arr = [1, 2, 3, 4, 5];
Span<int> middle = arr.AsSpan(1, 3);   // [2, 3, 4]를 가리킴 (복사 없음)
middle[0] = 99;
Console.WriteLine(arr[1]);             // 99 ← 원본이 바뀜

string date = "2026-11-23";
ReadOnlySpan<char> year = date.AsSpan(0, 4);   // 새 문자열 없이 "2026"
int y = int.Parse(year);
```

**도구 정리**

| 도구 | 하는 일 | 제한 |
| --- | --- | --- |
| `Span<T>`, `ReadOnlySpan<T>` | 연속된 메모리의 일부를 복사 없이 다룸 | 스택에만 있을 수 있음. 필드나 async 메서드에 못 둠 |
| `Memory<T>` | Span과 비슷하지만 힙에 둘 수 있음 | async에서 쓸 때 사용, 필요할 때 `.Span`으로 변환 |
| `ref struct` | 스택에만 있을 수 있는 구조체 (Span이 이것) | 박싱, 클래스 필드 불가 |
| `stackalloc` | 작은 배열을 힙이 아닌 스택에 만듦 | 작은 크기만 (보통 수백 바이트 이하) |
| `ArrayPool<T>` | 큰 배열을 빌려 쓰고 돌려줌 (재사용) | 다 쓰면 반드시 `Return` |

```csharp
Span<byte> buffer = stackalloc byte[256];   // GC가 신경 쓸 필요 없는 임시 버퍼
```

**언제 쓰나**

- 성능이 실제로 문제일 때만 쓴다. 먼저 `BenchmarkDotNet` 같은 도구로 측정하고, 느린 곳을 확인한 뒤에 바꾼다.
- 초보 단계에서는 읽기 쉬운 코드가 빠른 코드보다 중요하다.
- 이미 .NET의 많은 메서드(`int.Parse`, `string.Split` 등)가 내부에서 Span을 써서, 우리는 모르는 사이에 혜택을 받고 있다.

**연습**: `"홍길동,85,컴공"` 같은 CSV 한 줄을 `Split` 없이 `ReadOnlySpan<char>`과 `IndexOf(',')`만으로 나눠 점수를 정수로 꺼내 본다.

## 27장. 리플렉션과 특성(Attribute)

리플렉션은 **실행 중에 타입의 정보(이름, 속성, 메서드)를 조사하고 다루는** 기능이고, 특성은 코드에 붙이는 라벨(메타데이터)이다. 둘은 짝으로 쓰인다. 특성으로 라벨을 붙이고, 리플렉션으로 그 라벨을 읽어 동작을 바꾼다.

**특성: 이미 많이 보게 될 것**

```csharp
class User
{
    [Required]                         // 필수 입력 (검증 라이브러리가 읽음)
    [MaxLength(20)]
    public string Name { get; set; } = "";

    [JsonPropertyName("user_age")]     // JSON에서는 이 이름으로 (22장)
    public int Age { get; set; }

    [Obsolete("NewMethod를 쓰세요")]  // 쓰면 컴파일러가 경고
    public void OldMethod() { }
}
```

- 특성은 대괄호 `[ ]`로 붙이며, 그 자체로는 아무 일도 하지 않는다. 누군가(컴파일러, 프레임워크, 내 코드)가 읽어야 의미가 생긴다.
- ASP.NET Core의 `[HttpGet]`, 테스트의 `[Fact]`, Unity의 `[SerializeField]`가 모두 특성이다.

**리플렉션: 타입을 들여다보기**

```csharp
using System.Reflection;

Type t = typeof(User);              // 또는 객체.GetType()
Console.WriteLine(t.Name);          // User

foreach (PropertyInfo p in t.GetProperties())
    Console.WriteLine($"{p.Name}: {p.PropertyType.Name}");
// Name: String
// Age: Int32

// 특성 읽기
var prop = t.GetProperty("Name")!;
bool required = prop.GetCustomAttribute<RequiredAttribute>() != null;   // True
```

**나만의 특성 만들기**

```csharp
[AttributeUsage(AttributeTargets.Property)]
class DisplayNameAttribute(string text) : Attribute   // 이름은 ~Attribute로 끝냄
{
    public string Text { get; } = text;
}

class Product
{
    [DisplayName("상품명")] public string Name { get; set; } = "";   // 붙일 때는 Attribute 생략
    [DisplayName("가격")] public decimal Price { get; set; }
}
```

**주의할 점**

- 리플렉션은 느리고, 이름을 문자열로 다루기 때문에 오타를 컴파일러가 잡아주지 못한다.
- 일반 앱 코드에서는 직접 쓸 일이 거의 없고, 프레임워크 내부에서 쓰인다.
- 최근에는 리플렉션 대신 28장의 소스 생성기를 쓰는 추세이다.

**연습**: 아무 객체나 받아 모든 public 속성의 이름과 값을 출력하는 `PrintAll(object obj)` 메서드를 리플렉션으로 만든다. `[DisplayName]`이 있으면 그 이름으로 출력한다.

## 28장. 소스 생성기와 Native AOT

두 기술 모두 **실행 중에 하던 일을 컴파일할 때 미리 해 두는** 방향이다. 실행이 빨라지고, 시작 시간과 메모리 사용량이 줄어든다. 초보 단계에서는 직접 만들기보다 "이미 쓰고 있는" 경우가 많다.

**소스 생성기 (Source Generator)**

컴파일하는 동안 내 코드를 읽고 **C# 코드를 자동으로 만들어 추가**해 주는 프로그램이다. 반복적인 코드를 사람 대신 써 주고, 27장의 리플렉션을 대신한다.

```csharp
using System.Text.Json.Serialization;
using System.Text.RegularExpressions;

// JSON: 리플렉션 대신 컴파일 때 만든 코드로 직렬화
[JsonSerializable(typeof(Student))]
partial class AppJsonContext : JsonSerializerContext { }

string json = JsonSerializer.Serialize(new Student("철수", 85), AppJsonContext.Default.Student);

// 정규식: 컴파일 때 최적화된 코드를 만듦
partial class Validators
{
    [GeneratedRegex(@"^\d{3}-\d{4}-\d{4}$")]
    public static partial Regex Phone();
}
bool ok = Validators.Phone().IsMatch("010-1234-5678");
```

- `partial`(14장)이 여기서 쓰인다. 내가 꼴만 선언하면 생성기가 나머지 반쪽을 채운다.
- 만들어진 코드는 Visual Studio의 솔루션 탐색기 > 종속성 > 분석기에서 볼 수 있다.

**Native AOT**

보통 C# 프로그램은 중간 코드(IL)로 컴파일됐다가, 실행할 때 .NET 런타임이 기계어로 바꾼다(JIT). Native AOT는 **미리 기계어로 다 바꿔 두어 런타임 없이 바로 실행되는 파일 하나**를 만든다.

| | 일반 (JIT) | Native AOT |
| --- | --- | --- |
| 시작 속도 | 보통 | 매우 빠름 |
| 배포 | .NET 런타임 필요 | 파일 하나만 복사 |
| 제약 | 거의 없음 | 리플렉션·동적 코드 생성에 제한 |
| 어울리는 곳 | 대부분의 앱 | 커맨드라인 도구, 클라우드 함수, 컨테이너 |

```xml
<!-- 프로젝트 파일(.csproj)에 한 줄 추가 -->
<PublishAot>true</PublishAot>
```

그다음 `dotnet publish -c Release`로 배포하면 된다. 리플렉션을 쓰는 코드는 경고가 나므로, 위의 소스 생성기 방식으로 바꿔야 하는 경우가 많다. 두 기술이 함께 소개되는 이유이다.

**연습**: 1부의 메뉴형 콘솔 프로그램을 Native AOT로 배포해 보고, 일반 배포와 파일 크기와 시작 속도를 비교한다.

## 29장. unsafe 코드와 포인터

`unsafe`는 **C#의 안전장치(범위 검사, GC의 메모리 관리)를 끄고 C처럼 메모리 주소를 직접 다루는** 구역이다. 알아두면 좋지만, 일반 개발에서는 거의 쓸 일이 없고 쓰지 않는 것이 원칙이다. C 언어 수업에서 배운 포인터와 연결해서 보면 이해가 쉽다.

```csharp
// 프로젝트 파일에 <AllowUnsafeBlocks>true</AllowUnsafeBlocks> 필요
unsafe
{
    int x = 10;
    int* p = &x;          // x의 주소
    *p = 20;              // 주소를 따라가 값을 바꿈
    Console.WriteLine(x); // 20

    int[] arr = [1, 2, 3];
    fixed (int* ptr = arr)    // GC가 배열을 옮기지 못하게 고정
    {
        Console.WriteLine(*(ptr + 1));   // 2
    }
}
```

| 문법 | 의미 |
| --- | --- |
| `int*` | int를 가리키는 포인터 타입 |
| `&x` | x의 메모리 주소 |
| `*p` | 포인터가 가리키는 값 |
| `fixed` | GC가 객체를 옮기지 못하게 잠시 고정 |

**언제 쓰나**

- C/C++로 만든 라이브러리(네이티브 DLL)와 데이터를 주고받을 때
- 극한의 성능이 필요한 이미지 처리, 게임 엔진 내부
- 대부분의 경우 26장의 `Span<T>`이 안전하게 같은 일을 해 주므로, 먼저 Span으로 해결할 수 있는지 본다.

**주의할 점**

- 범위를 벗어나도 예외가 나지 않고 다른 메모리를 덮어쓴다. 프로그램이 엉뚱한 곳에서 죽거나 보안 취약점이 된다.
- C# 15부터는 unsafe를 "컴파일러가 검증할 수 없는 부분을 개발자가 책임지는 약속"으로 다시 설계하는 작업이 미리보기로 시작됐다. 문법이 바뀔 수 있는 영역이니 깊이 외울 필요는 없다.

**연습**: 포인터로 배열의 모든 요소를 두 배로 만드는 코드를 작성한 뒤, 같은 일을 `Span<int>`과 `foreach (ref var x in span)`으로 다시 작성해 비교해 본다.

## 30장. 의존성 주입과 설계 패턴

의존성 주입(DI)은 **클래스가 필요한 부품을 스스로 `new` 하지 않고, 밖에서 넣어 주도록** 만드는 설계 방식이다. 부품을 갈아 끼우기 쉬워져서 테스트와 유지보수가 편해진다. ASP.NET Core, MAUI 등 현대 .NET 프레임워크는 모두 이 방식으로 돌아간다.

**문제: 직접 new 하면**

```csharp
class OrderService
{
    private readonly EmailSender sender = new EmailSender();   // 이메일에 단단히 묶임
    public void Order() { /* ... */ sender.Send("주문 완료"); }
}
// 문자 알림으로 바꾸려면 OrderService를 고쳐야 하고,
// 테스트할 때마다 진짜 이메일이 나감
```

**해결: 인터페이스 + 생성자 주입**

```csharp
interface INotifier { void Send(string msg); }      // 11장 인터페이스

class EmailNotifier : INotifier { public void Send(string msg) => Console.WriteLine($"[메일] {msg}"); }
class SmsNotifier   : INotifier { public void Send(string msg) => Console.WriteLine($"[문자] {msg}"); }

class OrderService(INotifier notifier)              // 필요한 부품을 생성자로 받음
{
    public void Order() => notifier.Send("주문 완료");
}

var service = new OrderService(new SmsNotifier());  // 바깥에서 골라서 넣어줌
service.Order();
```

**DI 컨테이너: 조립을 자동으로**

부품이 많아지면 `new`를 일일이 하기 힘들다. DI 컨테이너에 "이 인터페이스에는 이 클래스"를 등록해 두면 알아서 조립해 준다.

```csharp
using Microsoft.Extensions.DependencyInjection;   // NuGet 패키지

var services = new ServiceCollection();
services.AddSingleton<INotifier, EmailNotifier>();   // 등록
services.AddTransient<OrderService>();

var provider = services.BuildServiceProvider();
var order = provider.GetRequiredService<OrderService>();   // EmailNotifier가 자동으로 들어감
order.Order();
```

| 수명(Lifetime) | 의미 |
| --- | --- |
| `AddSingleton` | 프로그램 전체에서 하나만 만들어 공유 |
| `AddScoped` | 요청(웹의 경우 HTTP 요청) 하나당 하나 |
| `AddTransient` | 필요할 때마다 새로 만듦 |

**자주 들을 설계 패턴**

설계 패턴은 자주 나오는 문제의 검증된 해결 방법에 붙인 이름이다. 외우기보다 "이런 이름이 있다"는 정도로 알아두고, 필요할 때 찾아보면 된다.

| 패턴 | 한 줄 설명 | C#에서 보는 곳 |
| --- | --- | --- |
| 싱글톤 | 객체를 하나만 만들어 공유 | `AddSingleton` |
| 팩토리 | 어떤 객체를 만들지 결정하는 일을 따로 맡김 | `HttpClientFactory` |
| 전략 | 바꿀 수 있는 알고리즘을 인터페이스로 분리 | 위의 `INotifier` |
| 옵저버 | 상태가 바뀌면 구독자에게 알림 | 17장 이벤트 |
| 리포지토리 | 데이터 저장/조회를 한 클래스에 모음 | EF Core와 함께 |

**연습**: 3부 가계부의 저장 기능을 `IEntryRepository` 인터페이스로 분리하고, `JsonEntryRepository`(파일 저장)와 `MemoryEntryRepository`(테스트용) 두 구현을 만든다. 주입하는 한 줄만 바꿔 전환되는지 확인한다.

## 4부 마무리

4부는 주제가 넓어서 프로젝트 하나로 다 담기 어렵다. 필수인 23장과 30장을 합친 프로젝트 하나만 해 본다.

- [ ] **날씨 조회 콘솔 앱**: `IWeatherService` 인터페이스와 공개 날씨 API를 부르는 구현을 만든다(`HttpClient`, async/await). 도시 여러 개를 `Task.WhenAll`로 동시에 조회하고, 5초가 넘으면 `CancellationToken`으로 취소한다. 서비스는 DI 컨테이너로 주입하고, 테스트용으로 고정된 값을 돌려주는 가짜 구현도 만든다.

**다음 단계: 5부 최신 기능과 부록**

5부에서는 C# 12~15에서 새로 생긴 문법을 한데 모아 정리하고, 부록에서 단위 테스트와 실제 앱 종류(콘솔, 웹, 모바일)를 맛본다.

---

[← 3부 중급](03-intermediate.md) · [목차](README.md) · [5부 최신 기능 + 부록 →](05-latest-and-appendix.md)
