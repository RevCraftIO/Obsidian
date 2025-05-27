---
作成日: 2025-05-27
tags:
  - CSharp
子文書:
---

# C# 13.0 新機能まとめ

C# 13.0では、パフォーマンスの向上とコードの表現力の強化に焦点を当てた新機能が追加されました。

## 1. Params Span

Span<T>をparamsパラメータとして使用できるようになりました。

```csharp
// params Spanの使用
public static int Sum(params Span<int> numbers)
{
    int sum = 0;
    foreach (var num in numbers)
    {
        sum += num;
    }
    return sum;
}

// 使用例
int[] array = [1, 2, 3, 4, 5];
Console.WriteLine(Sum(array));  // 15
Console.WriteLine(Sum(1, 2, 3)); // 6
```

## 2. Method Group Conversion to Delegate

メソッドグループからデリゲートへの変換が改善されました。

```csharp
// メソッドグループの変換
public class Calculator
{
    public static int Add(int x, int y) => x + y;
    public static int Subtract(int x, int y) => x - y;
}

// 使用例
Func<int, int, int> operation = Calculator.Add;
var result = operation(5, 3); // 8

// より複雑な例
var operations = new Dictionary<string, Func<int, int, int>>
{
    ["加算"] = Calculator.Add,
    ["減算"] = Calculator.Subtract
};
```

## 3. Auto Properties with Field Initializers

自動プロパティでフィールド初期化子を使用できます。

```csharp
public class Configuration
{
    public string Server { get; } = "localhost";
    public int Port { get; } = 8080;
    public bool IsSecure { get; } = true;
    
    // 計算された初期値
    public string ConnectionString { get; } = $"Server={Server};Port={Port}";
}

// 使用例
var config = new Configuration();
Console.WriteLine(config.ConnectionString); // Server=localhost;Port=8080
```

## 4. Pattern Matching on Span<char>

Span<char>に対するパターンマッチングが強化されました。

```csharp
// Span<char>のパターンマッチング
bool IsValidPhoneNumber(ReadOnlySpan<char> phone) => phone switch
{
    ['0', '9', '0', '-', .., '-', ..] => true,  // 090-XXXX-XXXX
    ['0', '8', '0', '-', .., '-', ..] => true,  // 080-XXXX-XXXX
    ['0', '7', '0', '-', .., '-', ..] => true,  // 070-XXXX-XXXX
    _ => false
};

// 使用例
string phone = "090-1234-5678";
Console.WriteLine(IsValidPhoneNumber(phone)); // True
```

## 5. Extended Fixed Statement

fixedステートメントの機能が拡張されました。

```csharp
// 拡張されたfixedステートメント
unsafe
{
    int[] array = [1, 2, 3, 4, 5];
    fixed (int* ptr = array)
    {
        // 配列の要素にアクセス
        for (int i = 0; i < array.Length; i++)
        {
            Console.WriteLine(*(ptr + i));
        }
    }
}
```

## 6. Improved Target-typed new Expressions

ターゲット型のnew式が改善されました。

```csharp
// 改善されたターゲット型のnew式
public class Person
{
    public string Name { get; }
    public int Age { get; }
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

// 使用例
Person person = new("田中太郎", 30);
List<Person> people = new() { new("鈴木一郎", 25), new("佐藤花子", 28) };
```

## 7. Enhanced Interpolated String Handling

補間文字列の処理が強化されました。

```csharp
// 強化された補間文字列
string name = "田中太郎";
int age = 30;

// 複雑な条件式を含む補間
string message = $"""
    ユーザー情報:
    名前: {name}
    年齢: {age}
    区分: {(age >= 20 ? "成人" : "未成年")}
    登録日: {DateTime.Now:yyyy-MM-dd}
    """;

// 式の補間
string result = $"計算結果: {Math.Pow(2, 3)}";
```

## 8. Improved Generic Type Inference

ジェネリック型の推論が改善されました。

```csharp
// 改善された型推論
public class Repository<T>
{
    public T Create<TValue>(TValue value) where TValue : T
    {
        return (T)Convert.ChangeType(value, typeof(T));
    }
}

// 使用例
var repo = new Repository<int>();
var number = repo.Create("123"); // 文字列からintへの変換
```

## 9. Enhanced Nullable Reference Types

null許容参照型の機能が強化されました。

```csharp
#nullable enable

public class User
{
    public required string Name { get; init; }
    public string? Email { get; init; }
    public required string PhoneNumber { get; init; }
    
    public string GetContactInfo() => Email ?? PhoneNumber;
}

// 使用例
var user = new User
{
    Name = "田中太郎",
    PhoneNumber = "090-1234-5678"
};
```

## 10. Improved Exception Handling

例外処理の機能が強化されました。

```csharp
// 強化された例外処理
public class DataProcessor
{
    public async Task ProcessDataAsync()
    {
        try
        {
            await LoadDataAsync();
        }
        catch (Exception ex) when (ex is IOException or UnauthorizedAccessException)
        {
            // ファイル関連の例外を処理
            LogError("ファイルアクセスエラー", ex);
        }
        catch (Exception ex) when (ex is TimeoutException)
        {
            // タイムアウト例外を処理
            LogError("タイムアウト", ex);
        }
    }
}
```

## まとめ

C# 13.0では以下の主要な改善がありました：

- **Params Span**: パフォーマンスの向上
- **Method Group Conversion**: デリゲートの使用が簡潔に
- **Auto Properties with Field Initializers**: プロパティ初期化の簡素化
- **Pattern Matching on Span<char>**: 文字列処理の効率化
- **Extended Fixed Statement**: ポインタ操作の安全性向上
- **Improved Target-typed new Expressions**: オブジェクト生成の簡潔化

これらの機能により、C#のコードはより効率的で安全になり、パフォーマンスも向上しました。特にParams SpanとPattern Matching on Span<char>は、パフォーマンスが重要な場面で大きな恩恵をもたらします。
