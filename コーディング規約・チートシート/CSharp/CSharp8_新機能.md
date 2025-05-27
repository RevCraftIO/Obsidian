---
作成日: 2025-05-27
tags:
  - CSharp
子文書:
---
# C# 8.0 新機能まとめ

C# 8.0では多くの新機能が追加され、コードの簡潔性、安全性、パフォーマンスが向上しました。

## 1. Nullable Reference Types

従来の参照型にnull許容性の概念を導入し、NullReferenceExceptionを防ぐのに役立ちます。

### 基本的な使用例

```csharp
#nullable enable

public class Person
{
    public string Name { get; set; }        // null許可しない
    public string? NickName { get; set; }   // null許可する
    
    public Person(string name)
    {
        Name = name;  // 必須
    }
}

// 使用例
var person = new Person("太郎");
person.NickName = null;  // OK
// person.Name = null;   // コンパイル警告
```

## 2. Switch Expressions

従来のswitch文をより簡潔に記述できる新しい構文です。

### 基本的な使用例

```csharp
// 従来のswitch文
string GetDayType(DayOfWeek day)
{
    return day switch
    {
        DayOfWeek.Monday => "月曜日",
        DayOfWeek.Tuesday => "火曜日",
        DayOfWeek.Saturday or DayOfWeek.Sunday => "週末",
        _ => "平日"
    };
}

// パターンマッチングとの組み合わせ
decimal CalculateDiscount(Customer customer) => customer switch
{
    { Type: CustomerType.Premium, Years: > 5 } => 0.2m,
    { Type: CustomerType.Premium } => 0.1m,
    { Years: > 10 } => 0.05m,
    _ => 0m
};
```

## 3. Pattern Matching Enhancements

パターンマッチング機能が大幅に拡張されました。

### Property Patterns

```csharp
public class Address
{
    public string Country { get; set; }
    public string State { get; set; }
}

public class Person
{
    public string Name { get; set; }
    public Address Address { get; set; }
}

// プロパティパターン
static string GetTaxRate(Person person) => person switch
{
    { Address: { Country: "Japan", State: "Tokyo" } } => "東京都税率",
    { Address: { Country: "Japan" } } => "日本の税率",
    { Address: { Country: "USA", State: "CA" } } => "カリフォルニア州税率",
    _ => "標準税率"
};
```

### Tuple Patterns

```csharp
// タプルパターン
public static string RockPaperScissors(string first, string second)
    => (first, second) switch
    {
        ("rock", "paper") => "Paper wins",
        ("rock", "scissors") => "Rock wins",
        ("paper", "rock") => "Paper wins",
        ("paper", "scissors") => "Scissors wins",
        ("scissors", "rock") => "Rock wins",
        ("scissors", "paper") => "Scissors wins",
        (_, _) => "Draw"
    };
```

### Positional Patterns

```csharp
public class Point
{
    public int X { get; set; }
    public int Y { get; set; }
    
    public void Deconstruct(out int x, out int y) => (x, y) = (X, Y);
}

static string Classify(Point point) => point switch
{
    (0, 0) => "原点",
    (var x, 0) => $"X軸上の点 ({x}, 0)",
    (0, var y) => $"Y軸上の点 (0, {y})",
    (var x, var y) when x == y => $"対角線上の点 ({x}, {y})",
    (var x, var y) => $"一般的な点 ({x}, {y})"
};
```

## 4. Using Declarations

usingステートメントをより簡潔に記述できます。

```csharp
// 従来の方法
public void ReadFile()
{
    using (var file = new FileStream("file.txt", FileMode.Open))
    {
        // ファイル処理
        // ここでファイルが自動的に閉じられる
    }
}

// C# 8の新しい方法
public void ReadFileNew()
{
    using var file = new FileStream("file.txt", FileMode.Open);
    // ファイル処理
    // メソッドの終了時に自動的にファイルが閉じられる
    
    // 複数のリソースも扱える
    using var reader = new StreamReader(file);
    using var writer = new StreamWriter("output.txt");
    
    // 処理...
} // ここで全てのリソースが自動的に解放される
```

## 5. Static Local Functions

ローカル関数にstaticキーワードを付けることができます。

```csharp
public int Calculate(int[] numbers)
{
    return ProcessNumbers();
    
    // staticローカル関数は外部変数をキャプチャできない
    static int ProcessNumbers()
    {
        // numbers配列にアクセスできない（コンパイルエラー）
        return 42;
    }
}

// パラメータを使用する例
public int CalculateSum(int[] numbers)
{
    return Sum(numbers);
    
    static int Sum(int[] nums)
    {
        int total = 0;
        foreach (int num in nums)
        {
            total += num;
        }
        return total;
    }
}
```

## 6. Readonly Members

structのメンバーを個別にreadonlyにできます。

```csharp
public struct Point3D
{
    public double X { get; set; }
    public double Y { get; set; }
    public double Z { get; set; }
    
    // readonlyメンバー
    public readonly double Distance => Math.Sqrt(X * X + Y * Y + Z * Z);
    
    public readonly override string ToString() => $"({X}, {Y}, {Z})";
    
    // 通常のメンバー（状態を変更可能）
    public void Reset()
    {
        X = Y = Z = 0;
    }
}
```

## 7. Default Interface Methods

インターフェースにデフォルト実装を提供できます。

```csharp
public interface ILogger
{
    void Log(string message);
    
    // デフォルト実装
    void LogError(string message) => Log($"ERROR: {message}");
    void LogWarning(string message) => Log($"WARNING: {message}");
    
    // 静的メンバーも定義可能
    static void LogInfo(string message) => Console.WriteLine($"INFO: {message}");
}

public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
    
    // LogErrorとLogWarningは自動的に使用可能
}

// 使用例
var logger = new ConsoleLogger();
logger.Log("通常のログ");
logger.LogError("エラーメッセージ");
logger.LogWarning("警告メッセージ");
```

## 8. Asynchronous Streams (IAsyncEnumerable)

非同期でデータを列挙できるようになりました。

```csharp
public static async IAsyncEnumerable<int> GenerateSequence()
{
    for (int i = 0; i < 20; i++)
    {
        await Task.Delay(100); // 非同期待機
        yield return i;
    }
}

// 使用例
public static async Task ConsumeAsyncStream()
{
    await foreach (var number in GenerateSequence())
    {
        Console.WriteLine(number);
    }
}

// 実際のデータベースアクセス例
public static async IAsyncEnumerable<Customer> GetCustomersAsync()
{
    using var connection = new SqlConnection(connectionString);
    await connection.OpenAsync();
    
    using var command = new SqlCommand("SELECT * FROM Customers", connection);
    using var reader = await command.ExecuteReaderAsync();
    
    while (await reader.ReadAsync())
    {
        yield return new Customer
        {
            Id = reader.GetInt32("Id"),
            Name = reader.GetString("Name")
        };
    }
}
```

## 9. Indices and Ranges

配列やコレクションのインデックス指定が便利になりました。

```csharp
var numbers = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// インデックス演算子
var lastItem = numbers[^1];        // 9 (最後の要素)
var secondLast = numbers[^2];      // 8 (最後から2番目)

// レンジ演算子
var firstThree = numbers[0..3];    // { 0, 1, 2 }
var lastThree = numbers[^3..];     // { 7, 8, 9 }
var middle = numbers[2..^2];       // { 2, 3, 4, 5, 6, 7 }
var all = numbers[..];             // 全要素のコピー

// 文字列でも使用可能
string text = "Hello, World!";
var firstChar = text[0];           // 'H'
var lastChar = text[^1];           // '!'
var greeting = text[0..5];         // "Hello"
var world = text[7..^1];           // "World"
```

## 10. Null-coalescing Assignment

null合体代入演算子により、null チェックと代入を同時に行えます。

```csharp
// 従来の方法
if (name == null)
{
    name = "デフォルト名";
}

// C# 8の新しい方法
name ??= "デフォルト名";

// より複雑な例
public class ConfigurationManager
{
    private Dictionary<string, string> _cache;
    
    public string GetValue(string key)
    {
        _cache ??= new Dictionary<string, string>();
        
        if (!_cache.TryGetValue(key, out string value))
        {
            value = LoadFromDatabase(key);
            _cache[key] = value;
        }
        
        return value;
    }
    
    private string LoadFromDatabase(string key) => "database_value";
}
```

## 11. Unmanaged Constructed Types

ジェネリック型でもunmanagedとして扱えるようになりました。

```csharp
// unmanagedジェネリック構造体
public struct Coordinates<T> where T : unmanaged
{
    public T X;
    public T Y;
    
    public Coordinates(T x, T y)
    {
        X = x;
        Y = y;
    }
}

// 使用例
unsafe void UseUnmanagedGenerics()
{
    var intCoords = new Coordinates<int>(10, 20);
    var doubleCoords = new Coordinates<double>(1.5, 2.5);
    
    // ポインタとして使用可能
    fixed (Coordinates<int>* ptr = &intCoords)
    {
        // ポインタ操作
        Console.WriteLine($"X: {ptr->X}, Y: {ptr->Y}");
    }
}
```

## 12. Stackalloc in Nested Expressions

stackallocをより柔軟に使用できるようになりました。

```csharp
// 従来は変数宣言でのみ使用可能だった
// Span<int> numbers = stackalloc int[3] { 1, 2, 3 };

// C# 8では式の中でも使用可能
public int CalculateSum()
{
    return AddAll(stackalloc int[3] { 1, 2, 3 });
}

static int AddAll(Span<int> numbers)
{
    int sum = 0;
    foreach (int num in numbers)
    {
        sum += num;
    }
    return sum;
}

// 三項演算子での使用
public Span<int> GetNumbers(bool useSmallArray)
{
    return useSmallArray 
        ? stackalloc int[3] { 1, 2, 3 }
        : stackalloc int[5] { 1, 2, 3, 4, 5 };
}
```

## まとめ

C# 8.0では以下の主要な改善がありました：

- **Nullable Reference Types**: null安全性の向上
- **Switch Expressions**: より簡潔な条件分岐
- **Pattern Matching**: 強力なパターンマッチング機能
- **Using Declarations**: リソース管理の簡素化
- **Asynchronous Streams**: 非同期データストリーミング
- **Indices and Ranges**: 配列操作の利便性向上

これらの機能により、C#のコードはより安全で、簡潔で、パフォーマンスが良くなりました。特にNullable Reference TypesとPattern Matchingは、日常的な開発において大きな恩恵をもたらします。