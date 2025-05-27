---
作成日: 2025-05-27
tags:
  - CSharp
子文書:
---
# C# 11.0 新機能まとめ

C# 11.0では生産性とパフォーマンスの向上に焦点を当てた多くの新機能が追加されました。

## 1. Raw String Literals

複数行の文字列や特殊文字を含む文字列を簡潔に記述できます。

### 基本的な使用例

```csharp
// 従来の方法
string json = "{\n  \"name\": \"太郎\",\n  \"age\": 30\n}";
string path = "C:\\Users\\Username\\Documents\\file.txt";

// Raw String Literalsを使用
string jsonRaw = """
{
  "name": "太郎",
  "age": 30
}
""";

string pathRaw = """C:\Users\Username\Documents\file.txt""";

// SQLクエリの例
string sqlQuery = """
    SELECT u.Name, u.Email, p.Title
    FROM Users u
    INNER JOIN Posts p ON u.Id = p.UserId
    WHERE u.IsActive = 1
    AND p.PublishDate >= '2023-01-01'
    ORDER BY p.PublishDate DESC
    """;
```

### 補間を含むRaw String Literals

```csharp
string userName = "田中太郎";
int userAge = 30;

// 補間を含むRaw String Literals
string userInfo = $$"""
{
  "user": {
    "name": "{{userName}}",
    "age": {{userAge}},
    "profile": {
      "description": "これは{{userName}}さんのプロフィールです"
    }
  }
}
""";

// HTML生成の例
string title = "マイページ";
string content = "ようこそ！";

string html = $$"""
<!DOCTYPE html>
<html>
<head>
    <title>{{title}}</title>
</head>
<body>
    <h1>{{title}}</h1>
    <p>{{content}}</p>
</body>
</html>
""";
```

## 2. Required Members

プロパティやフィールドを必須として指定できます。

### 基本的な使用例

```csharp
public class Person
{
    public required string Name { get; init; }
    public required int Age { get; init; }
    public string? Email { get; init; }
}

// 使用例
var person = new Person
{
    Name = "田中太郎",  // 必須
    Age = 30,          // 必須
    Email = "tanaka@example.com"  // 任意
};

// Name または Age を省略するとコンパイルエラー
// var invalidPerson = new Person { Email = "test@example.com" }; // エラー
```

### SetsRequiredMembersとの組み合わせ

```csharp
public class Product
{
    public required string Name { get; init; }
    public required decimal Price { get; init; }
    public required string Category { get; init; }
    public DateTime CreatedAt { get; init; } = DateTime.Now;
    
    [SetsRequiredMembers]
    public Product(string name, decimal price, string category)
    {
        Name = name;
        Price = price;
        Category = category;
    }
    
    // デフォルトコンストラクターを使用する場合は required プロパティの設定が必要
    public Product() { }
}

// 使用例
var product1 = new Product("ノートPC", 89800, "コンピュータ"); // OK
var product2 = new Product  // OK
{
    Name = "マウス",
    Price = 2980,
    Category = "周辺機器"
};
```

## 3. Generic Attributes

属性でジェネリック型を使用できるようになりました。

```csharp
// ジェネリック属性の定義
[AttributeUsage(AttributeTargets.Method)]
public class ExpectedResultAttribute<T> : Attribute
{
    public T ExpectedValue { get; }
    
    public ExpectedResultAttribute(T expectedValue)
    {
        ExpectedValue = expectedValue;
    }
}

// 使用例
public class Calculator
{
    [ExpectedResult<int>(10)]
    public int Add(int a, int b) => a + b;
    
    [ExpectedResult<string>("Hello World")]
    public string Concatenate(string a, string b) => a + " " + b;
    
    [ExpectedResult<bool>(true)]
    public bool IsPositive(int number) => number > 0;
}

// より実用的な例
[AttributeUsage(AttributeTargets.Property)]
public class ValidRangeAttribute<T> : Attribute where T : IComparable<T>
{
    public T MinValue { get; }
    public T MaxValue { get; }
    
    public ValidRangeAttribute(T minValue, T maxValue)
    {
        MinValue = minValue;
        MaxValue = maxValue;
    }
}

public class UserProfile
{
    [ValidRange<int>(0, 150)]
    public int Age { get; set; }
    
    [ValidRange<decimal>(0, 1000000)]
    public decimal Salary { get; set; }
    
    [ValidRange<DateTime>(typeof(DateTime), "2000-01-01", "2030-12-31")]
    public DateTime BirthDate { get; set; }
}
```

## 4. Newlines in String Interpolations

文字列補間内で改行を使用できるようになりました。

```csharp
string name = "田中太郎";
int age = 30;
string department = "開発部";

// 複雑な条件式を改行して記述
string message = $"ユーザー情報: {(age >= 18 
    ? $"{name}さん（{age}歳）は成人です" 
    : $"{name}さん（{age}歳）は未成年です")}";

// LINQクエリを補間内で使用
var numbers = new[] { 1, 2, 3, 4, 5 };
string result = $"偶数の合計: {numbers
    .Where(x => x % 2 == 0)
    .Sum()}";

// より複雑な例
var users = new[]
{
    new { Name = "太郎", Age = 25, Department = "開発" },
    new { Name = "花子", Age = 30, Department = "営業" },
    new { Name = "次郎", Age = 35, Department = "開発" }
};

string summary = $"""
開発部メンバー:
{string.Join("\n", users
    .Where(u => u.Department == "開発")
    .Select(u => $"  - {u.Name} ({u.Age}歳)"))}

平均年齢: {users
    .Where(u => u.Department == "開発")
    .Average(u => u.Age):F1}歳
""";
```

## 5. List Patterns

リストやコレクションに対するパターンマッチングができます。

```csharp
// 基本的なリストパターン
int CheckNumbers(int[] numbers) => numbers switch
{
    [] => 0,                                    // 空の配列
    [var single] => single,                     // 要素が1つ
    [var first, var second] => first + second,  // 要素が2つ
    [var first, .., var last] => first + last,  // 最初と最後
    [1, 2, 3] => 100,                          // 特定の値
    [1, ..] => 200,                            // 1で始まる
    [.., 9] => 300,                            // 9で終わる
    _ => -1
};

// より実用的な例
string ProcessCommand(string[] args) => args switch
{
    ["help"] => "ヘルプを表示します",
    ["version"] => "バージョン 1.0.0",
    ["create", var fileName] => $"ファイル '{fileName}' を作成します",
    ["delete", var fileName] => $"ファイル '{fileName}' を削除します",
    ["copy", var source, var dest] => $"'{source}' を '{dest}' にコピーします",
    ["list", ..] => "ファイル一覧を表示します",
    [] => "コマンドを指定してください",
    _ => "不明なコマンドです"
};

// コレクションでの使用
public record LogEntry(DateTime Time, string Level, string Message);

string AnalyzeLogs(List<LogEntry> logs) => logs switch
{
    [] => "ログがありません",
    [var single] => $"ログが1件あります: {single.Message}",
    [.., { Level: "ERROR" }] => "最後のログがエラーです",
    [{ Level: "ERROR" }, ..] => "最初のログがエラーです",
    var allLogs when allLogs.All(l => l.Level == "INFO") => "すべて情報ログです",
    _ => $"ログが{logs.Count}件あります"
};
```

## 6. UTF-8 String Literals

UTF-8文字列リテラルを直接定義できます。

```csharp
// UTF-8文字列リテラル
ReadOnlySpan<byte> utf8String = "Hello, 世界!"u8;
byte[] utf8Bytes = "こんにちは"u8.ToArray();

// JSONデータの処理例
ReadOnlySpan<byte> jsonData = """
{
  "name": "田中太郎",
  "age": 30,
  "city": "東京"
}
"""u8;

// Web APIでの使用例
public class ApiController
{
    public async Task<IActionResult> GetUserAsync(int id)
    {
        // UTF-8文字列として直接レスポンスを生成
        var response = $$"""
        {
          "id": {{id}},
          "message": "ユーザー情報を取得しました",
          "timestamp": "{{DateTime.Now:yyyy-MM-ddTHH:mm:ss}}"
        }
        """u8;
        
        return new ContentResult
        {
            Content = System.Text.Encoding.UTF8.GetString(response),
            ContentType = "application/json"
        };
    }
}
```

## 7. Static Abstract Members in Interfaces

インターフェースで静的抽象メンバーを定義できます。

```csharp
// 数値型のインターフェース
public interface INumber<T> where T : INumber<T>
{
    static abstract T Zero { get; }
    static abstract T One { get; }
    static abstract T operator +(T left, T right);
    static abstract T operator -(T left, T right);
    static abstract T Parse(string s);
}

// 実装例
public struct MyNumber : INumber<MyNumber>
{
    private readonly int _value;
    
    public MyNumber(int value) => _value = value;
    
    public static MyNumber Zero => new(0);
    public static MyNumber One => new(1);
    
    public static MyNumber operator +(MyNumber left, MyNumber right)
        => new(left._value + right._value);
    
    public static MyNumber operator -(MyNumber left, MyNumber right)
        => new(left._value - right._value);
    
    public static MyNumber Parse(string s) => new(int.Parse(s));
    
    public override string ToString() => _value.ToString();
}

// ジェネリック数学関数
public static class MathUtils
{
    public static T Sum<T>(params T[] values) where T : INumber<T>
    {
        T result = T.Zero;
        foreach (T value in values)
        {
            result = result + value;
        }
        return result;
    }
    
    public static T Average<T>(params T[] values) where T : INumber<T>
    {
        if (values.Length == 0) return T.Zero;
        return Sum(values); // 簡略化された例
    }
}

// 使用例
var numbers = new MyNumber[] { new(1), new(2), new(3), new(4), new(5) };
var sum = MathUtils.Sum(numbers);
Console.WriteLine($"合計: {sum}"); // 合計: 15
```

## 8. Auto-default Structs

構造体のフィールドが自動的にデフォルト値で初期化されます。

```csharp
// 従来は全てのフィールドを初期化する必要があった
public struct Point
{
    public int X;
    public int Y;
    public int Z;
    
    // C# 11では一部のフィールドのみ初期化すればよい
    public Point(int x, int y)
    {
        X = x;
        Y = y;
        // Z は自動的に 0 で初期化される
    }
}

// より複雑な例
public struct Rectangle
{
    public int Width;
    public int Height;
    public string Name;
    public bool IsVisible;
    
    public Rectangle(int width, int height)
    {
        Width = width;
        Height = height;
        // Name は null、IsVisible は false で自動初期化
    }
    
    public Rectangle(string name)
    {
        Name = name;
        // Width、Height は 0、IsVisible は false で自動初期化
    }
}
```

## 9. Pattern Match Span<char> on String

文字列に対してSpan<char>のパターンマッチングができます。

```csharp
// 文字列の先頭部分のマッチング
bool StartsWithProtocol(string url) => url switch
{
    ['h', 't', 't', 'p', ':', '/', '/', ..] => true,
    ['h', 't', 't', 'p', 's', ':', '/', '/', ..] => true,
    ['f', 't', p', ':', '/', '/', ..] => true,
    _ => false
};

// ファイル拡張子の判定
string GetFileType(string fileName) => fileName switch
{
    [.., '.', 't', 'x', 't'] => "テキストファイル",
    [.., '.', 'j', 'p', 'g'] or [.., '.', 'j', 'p', 'e', 'g'] => "画像ファイル",
    [.., '.', 'p', 'd', 'f'] => "PDFファイル",
    [.., '.', 'z', 'i', 'p'] => "圧縮ファイル",
    _ => "不明なファイル"
};

// より実用的な例
public static class StringValidator
{
    public static bool IsValidEmail(string email) => email switch
    {
        [.., '@', .., '.', ..] when email.Count(c => c == '@') == 1 => true,
        _ => false
    };
    
    public static bool IsJapanesePhoneNumber(string phone) => phone switch
    {
        ['0', '9', '0', '-', _, _, _, _, '-', _, _, _, _] => true,  // 090-XXXX-XXXX
        ['0', '8', '0', '-', _, _, _, _, '-', _, _, _, _] => true,  // 080-XXXX-XXXX
        ['0', '7', '0', '-', _, _, _, _, '-', _, _, _, _] => true,  // 070-XXXX-XXXX
        _ => false
    };
}
```

## 10. Extended nameof Scope

nameof式でより多くのスコープにアクセスできます。

```csharp
// メソッドパラメータでの使用
public class UserService
{
    public void UpdateUser(int userId, string userName)
    {
        // パラメータ名を文字列として取得
        LogMethodCall(nameof(UpdateUser), nameof(userId), nameof(userName));
        
        // 実装...
    }
    
    private void LogMethodCall(string methodName, params string[] parameterNames)
    {
        Console.WriteLine($"メソッド {methodName} が呼び出されました");
        Console.WriteLine($"パラメータ: {string.Join(", ", parameterNames)}");
    }
}

// 属性での使用
public class ValidationAttribute : Attribute
{
    public string PropertyName { get; }
    
    public ValidationAttribute([CallerMemberName] string propertyName = "")
    {
        PropertyName = propertyName;
    }
}

public class Person
{
    [Validation] // PropertyName は "Name" になる
    public string Name { get; set; }
    
    [Validation] // PropertyName は "Age" になる  
    public int Age { get; set; }
}

// ローカル関数での使用
public void ProcessData()
{
    var data = GetData();
    
    void ValidateData()
    {
        if (data == null)
        {
            throw new ArgumentNullException(nameof(data)); // ローカル変数を参照
        }
    }
    
    ValidateData();
    
    // 処理...
}
```

## 11. Numeric IntPtr

IntPtrとUIntPtrで数値演算ができるようになりました。

```csharp
// 基本的な数値演算
nint pointer1 = 100;
nint pointer2 = 200;
nint result = pointer1 + pointer2; // 300

// ポインタ演算の例
unsafe
{
    int[] array = { 1, 2, 3, 4, 5 };
    fixed (int* ptr = array)
    {
        nint address = (nint)ptr;
        
        // ポインタの移動
        nint nextAddress = address + sizeof(int);
        nint offsetAddress = address + (sizeof(int) * 2);
        
        Console.WriteLine($"基準アドレス: 0x{address:X}");
        Console.WriteLine($"次のアドレス: 0x{nextAddress:X}");
        Console.WriteLine($"オフセットアドレス: 0x{offsetAddress:X}");
    }
}

// メモリサイズの計算
public static class MemoryCalculator
{
    public static nint CalculateArraySize<T>(int length) where T : unmanaged
    {
        return length * sizeof(T);
    }
    
    public static nint AlignToPageSize(nint size)
    {
        const nint pageSize = 4096;
        return (size + pageSize - 1) & ~(pageSize - 1);
    }
}
```

## まとめ

C# 11.0では以下の主要な改善がありました：

- **Raw String Literals**: 複数行文字列の簡潔な記述
- **Required Members**: 必須プロパティの指定
- **Generic Attributes**: 属性でのジェネリック型サポート
- **List Patterns**: コレクションのパターンマッチング
- **UTF-8 String Literals**: UTF-8文字列の直接定義
- **Static Abstract Members**: インターフェースの静的抽象メンバー

これらの機能により、C#のコードはより表現力豊かで、型安全性が向上し、パフォーマンスも改善されました。特にRaw String LiteralsとList Patternsは日常的な開発で大きな恩恵をもたらします。