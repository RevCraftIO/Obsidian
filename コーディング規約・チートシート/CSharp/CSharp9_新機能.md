---
作成日: 2025-05-27
tags:
  - CSharp
子文書:
---
# CSharp 9.0 新機能まとめ

CSharp 9.0では特にレコード型とパターンマッチングに焦点を当てた多くの新機能が追加され、関数型プログラミングのサポートが強化されました。

## 1. Records

不変データ構造を簡単に定義できる新しい型です。

### 基本的なRecord

```csharp
// 基本的なレコード定義
public record Person(string FirstName, string LastName);

// 使用例
var person1 = new Person("太郎", "田中");
var person2 = new Person("太郎", "田中");

Console.WriteLine(person1 == person2);        // True (値による比較)
Console.WriteLine(person1.FirstName);         // 太郎
Console.WriteLine(person1);                   // Person { FirstName = 太郎, LastName = 田中 }
```

### with式によるコピーと変更

```csharp
var originalPerson = new Person("太郎", "田中");
var modifiedPerson = originalPerson with { FirstName = "花子" };

Console.WriteLine(originalPerson);   // Person { FirstName = 太郎, LastName = 田中 }
Console.WriteLine(modifiedPerson);   // Person { FirstName = 花子, LastName = 田中 }
```

### 高度なRecord機能

```csharp
// プロパティとメソッドを追加したレコード
public record Employee(string FirstName, string LastName, decimal Salary)
{
    public string FullName => $"{FirstName} {LastName}";
    
    public Employee GiveRaise(decimal amount) => this with { Salary = Salary + amount };
    
    // 計算プロパティ
    public string SalaryCategory => Salary switch
    {
        < 30000 => "Junior",
        < 60000 => "Mid-level",
        _ => "Senior"
    };
}

// 使用例
var employee = new Employee("太郎", "田中", 45000);
Console.WriteLine(employee.FullName);          // 太郎 田中
Console.WriteLine(employee.SalaryCategory);    // Mid-level

var promotedEmployee = employee.GiveRaise(10000);
Console.WriteLine(promotedEmployee.Salary);    // 55000
```

### 継承可能なRecord

```csharp
public record Person(string FirstName, string LastName);
public record Student(string FirstName, string LastName, string School) : Person(FirstName, LastName);
public record Teacher(string FirstName, string LastName, string Subject) : Person(FirstName, LastName);

// 使用例
var student = new Student("太郎", "田中", "東京大学");
var teacher = new Teacher("花子", "佐藤", "数学");

// パターンマッチングでの使用
string GetDescription(Person person) => person switch
{
    Student s => $"{s.FirstName}さんは{s.School}の学生です",
    Teacher t => $"{t.FirstName}さんは{t.Subject}の先生です",
    _ => $"{person.FirstName}さんです"
};
```

## 2. Init-only Properties

オブジェクト初期化時のみ設定可能なプロパティです。

```csharp
public class ImmutablePerson
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
    public DateTime BirthDate { get; init; }
    
    public int Age => DateTime.Now.Year - BirthDate.Year;
}

// 使用例
var person = new ImmutablePerson
{
    FirstName = "太郎",
    LastName = "田中",
    BirthDate = new DateTime(1990, 5, 15)
};

Console.WriteLine(person.Age);
// person.FirstName = "花子"; // コンパイルエラー
```

### required修飾子との組み合わせ（CSharp 11で導入されましたが、概念はCSharp 9から）

```csharp
public class Product
{
    public string Name { get; init; }
    public decimal Price { get; init; }
    public string? Description { get; init; }
    
    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}

// オブジェクト初期化子での使用
var product = new Product("ノートPC", 89800)
{
    Description = "高性能なノートパソコン"
};
```

## 3. Top-level Programs

Mainメソッドを省略してより簡潔にプログラムを記述できます。

```csharp
// 従来の方法
using System;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}

// CSharp 9の新しい方法（Program.cs）
using System;

Console.WriteLine("Hello World!");
Console.WriteLine($"引数の数: {args.Length}");

foreach (var arg in args)
{
    Console.WriteLine($"引数: {arg}");
}

// 関数も定義可能
void PrintGreeting(string name)
{
    Console.WriteLine($"こんにちは、{name}さん！");
}

PrintGreeting("太郎");
```

## 4. Pattern Matching Enhancements

パターンマッチング機能がさらに強化されました。

### Relational Patterns

```csharp
// 関係演算子を使ったパターン
string GetAgeCategory(int age) => age switch
{
    < 13 => "子供",
    >= 13 and < 20 => "ティーンエイジャー",
    >= 20 and < 65 => "大人",
    >= 65 => "シニア"
};

// より複雑な例
string GetTemperatureDescription(double temp) => temp switch
{
    < 0 => "氷点下",
    >= 0 and < 10 => "寒い",
    >= 10 and < 20 => "涼しい",
    >= 20 and < 30 => "暖かい",
    >= 30 => "暑い"
};
```

### Logical Patterns

```csharp
// 論理演算子を使ったパターン
bool IsWeekend(DayOfWeek day) => day is DayOfWeek.Saturday or DayOfWeek.Sunday;

bool IsBusinessHour(DateTime dateTime) => dateTime.Hour is >= 9 and <= 17 
    and dateTime.DayOfWeek is not (DayOfWeek.Saturday or DayOfWeek.Sunday);

// 複雑な条件の組み合わせ
string GetPricing(Customer customer, DateTime date) => (customer.Type, date.DayOfWeek) switch
{
    (CustomerType.Premium, DayOfWeek.Saturday or DayOfWeek.Sunday) => "Weekend Premium Rate",
    (CustomerType.Premium, _) => "Premium Rate",
    (CustomerType.Regular, DayOfWeek.Saturday or DayOfWeek.Sunday) => "Weekend Rate",
    _ => "Standard Rate"
};
```

### Negated Patterns

```csharp
// not パターン
bool IsNotNull(object obj) => obj is not null;

string ProcessValue(object value) => value switch
{
    not null and not string => "null以外の非文字列",
    not null and string s when s.Length > 0 => $"文字列: {s}",
    _ => "nullまたは空文字列"
};
```

## 5. Target-typed new Expressions

型推論により、newキーワードの後の型名を省略できます。

```csharp
// 従来の方法
Dictionary<string, List<int>> data = new Dictionary<string, List<int>>();
List<string> names = new List<string>();

// CSharp 9の新しい方法
Dictionary<string, List<int>> data = new();
List<string> names = new();

// メソッドの引数でも使用可能
void ProcessList(List<string> items) { }

ProcessList(new() { "item1", "item2", "item3" });

// プロパティでも使用可能
public class Configuration
{
    public Dictionary<string, string> Settings { get; set; } = new();
    public List<string> AllowedHosts { get; set; } = new();
}

// 配列やSpanでの使用
int[] numbers = new[] { 1, 2, 3, 4, 5 };
Span<int> span = new(numbers);
```

## 6. Covariant Return Types

オーバーライドするメソッドでより派生した型を返すことができます。

```csharp
public abstract class Animal
{
    public abstract Animal Clone();
}

public class Dog : Animal
{
    public string Breed { get; set; }
    
    // Animalの代わりにDogを返すことができる
    public override Dog Clone()
    {
        return new Dog { Breed = this.Breed };
    }
}

public class Cat : Animal
{
    public bool IsIndoor { get; set; }
    
    public override Cat Clone()
    {
        return new Cat { IsIndoor = this.IsIndoor };
    }
}

// 使用例
Dog originalDog = new Dog { Breed = "Golden Retriever" };
Dog clonedDog = originalDog.Clone(); // 型キャストが不要
```

## 7. Target-typed Conditional Expressions

条件演算子の型推論が改善されました。

```csharp
// 従来は型を明示する必要があった場合がある
Person person = GetPerson();
string name = person != null ? person.Name : (string)null;

// CSharp 9では型推論が改善
Person person = GetPerson();
string name = person?.Name ?? null;

// より複雑な例
public T GetDefault<T>() where T : class
{
    return typeof(T) == typeof(string) ? (T)(object)"" : null;
}

// Linq での使用例
var items = new[] { 1, 2, 3, 4, 5 };
var result = items.Select(x => x > 3 ? new { Value = x, IsHigh = true } 
                                      : new { Value = x, IsHigh = false });
```

## 8. Static Anonymous Functions

ラムダ式と匿名関数にstaticキーワードを付けることができます。

```csharp
// 外部変数をキャプチャしないことを明示
Func<int, int, int> add = static (x, y) => x + y;

// LINQでの使用
var numbers = new[] { 1, 2, 3, 4, 5 };
var doubled = numbers.Select(static x => x * 2);

// より複雑な例
public class Calculator
{
    private int multiplier = 10;
    
    public IEnumerable<int> ProcessNumbers(IEnumerable<int> numbers)
    {
        // 外部変数をキャプチャしない（パフォーマンス向上）
        var processed = numbers.Select(static x => x * 2);
        
        // 外部変数をキャプチャする場合はstaticを付けられない
        var multiplied = numbers.Select(x => x * multiplier);
        
        return processed.Concat(multiplied);
    }
}
```

## 9. Module Initializers

アセンブリ読み込み時に実行される初期化コードを定義できます。

```csharp
using System.Runtime.CompilerServices;

internal class ModuleInitializer
{
    [ModuleInitializer]
    public static void Initialize()
    {
        Console.WriteLine("モジュールが初期化されました");
        // ログ設定、グローバル設定の初期化など
        SetupLogging();
        ConfigureGlobalSettings();
    }
    
    private static void SetupLogging()
    {
        // ログ設定
    }
    
    private static void ConfigureGlobalSettings()
    {
        // グローバル設定
    }
}
```

## 10. Partial Methods の拡張

partial メソッドでアクセス修飾子と戻り値を指定できるようになりました。

```csharp
// ファイル1: Person.Designer.cs
public partial class Person
{
    private string _name;
    
    public string Name
    {
        get => _name;
        set
        {
            _name = value;
            OnNameChanged(value);
        }
    }
    
    // 実装が必要なpartialメソッド
    public partial void OnNameChanged(string newName);
    
    // 実装が任意のpartialメソッド
    partial void OnNameChanging(string newName);
}

// ファイル2: Person.cs
public partial class Person
{
    public partial void OnNameChanged(string newName)
    {
        Console.WriteLine($"名前が{newName}に変更されました");
        // イベント発火、検証など
    }
    
    // OnNameChangingは実装しなくても良い（空の実装が自動生成される）
}
```

## 11. Native Integers (nint and nuint)

プラットフォーム固有のサイズを持つ整数型です。

```csharp
// 32ビットプラットフォームでは32ビット、64ビットプラットフォームでは64ビット
nint platformInt = 42;
nuint platformUInt = 42u;

// ポインタサイズと同じサイズ
unsafe
{
    int value = 100;
    int* ptr = &value;
    nint address = (nint)ptr;
    
    Console.WriteLine($"Address: 0x{address:X}");
}

// 配列のインデックスやサイズに最適
public class LargeArray<T>
{
    private T[] _items;
    
    public T this[nint index]
    {
        get => _items[index];
        set => _items[index] = value;
    }
    
    public nint Length => _items.Length;
}
```

## 12. Function Pointers

関数ポインタのサポートが追加されました（unsafeコンテキスト）。

```csharp
unsafe
{
    // 関数ポインタの宣言
    delegate*<int, int, int> addPtr = &Add;
    delegate*<string, void> printPtr = &Print;
    
    // 関数ポインタの使用
    int result = addPtr(5, 3);
    printPtr("Hello from function pointer!");
    
    Console.WriteLine($"Result: {result}");
}

static int Add(int a, int b) => a + b;
static void Print(string message) => Console.WriteLine(message);

// より高度な使用例
unsafe class FunctionPointerExample
{
    public static void ExecuteOperation(delegate*<int, int, int> operation, int a, int b)
    {
        int result = operation(a, b);
        Console.WriteLine($"結果: {result}");
    }
    
    public static void Demo()
    {
        ExecuteOperation(&Add, 10, 5);
        ExecuteOperation(&Multiply, 10, 5);
    }
    
    static int Multiply(int a, int b) => a * b;
}
```

## まとめ

CSharp 9.0では以下の主要な改善がありました：

- **Records**: 不変データ構造の簡単な定義
- **Init-only Properties**: 初期化時のみ設定可能なプロパティ
- **Top-level Programs**: より簡潔なプログラム記述
- **Pattern Matching Enhancements**: 関係演算子と論理演算子のサポート
- **Target-typed new**: 型推論によるnew式の簡素化
- **Covariant Return Types**: オーバーライド時の柔軟な戻り値型

これらの機能により、CSharpのコードはより関数型プログラミングに親和性が高く、簡潔で表現力豊かになりました。特にRecordsとパターンマッチングの強化は、データ中心のアプリケーション開発において大きな恩恵をもたらします。