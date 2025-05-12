## 目次

1. [[#CSharp 10の新機能]]
2. [[#CSharp 11の新機能]]
3. [[#CSharp 12の新機能]]
4. [[#CSharp 13の新機能]]

## CSharp 10の新機能

### 1. グローバル using ディレクティブ

プロジェクト全体で共通して使用する名前空間を一度に定義できます。

```csharp
// GlobalUsings.cs ファイルで定義
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
global using Microsoft.Extensions.Logging;
```

### 2. ファイルスコープの名前空間宣言

ネストレベルを減らし、インデントを減らすことができる構文です。

```csharp
// 旧スタイル
namespace MyCompany.MyProduct
{
    public class MyClass
    {
        // クラスの実装
    }
}

// 新スタイル (CSharp 10)
namespace MyCompany.MyProduct;

public class MyClass
{
    // クラスの実装
}
```

### 3. レコード構造体

値型として動作するレコードを定義できます。

```csharp
// レコードクラス (参照型)
public record Person(string FirstName, string LastName);

// レコードストラクト (値型)
public readonly record struct Point(double X, double Y);

// レコードストラクトの使用例
public class GeometryCalculator
{
    public double CalculateDistance(Point p1, Point p2)
    {
        return Math.Sqrt(Math.Pow(p2.X - p1.X, 2) + Math.Pow(p2.Y - p1.Y, 2));
    }
}
```

### 4. 構造体の改善

構造体でパラメータなしコンストラクタや初期化子を定義できるようになりました。

```csharp
public struct Temperature
{
    public double Celsius { get; set; }
    public double Fahrenheit => Celsius * 9 / 5 + 32;

    // CSharp 10で可能になったパラメータなしコンストラクタ
    public Temperature()
    {
        Celsius = 23.0; // デフォルト値
    }
}
```

### 5. ラムダ式の型推論の改善

ラムダ式に明示的な戻り値の型を指定できます。

```csharp
// 戻り値の型を明示的に指定
var factorial = int (int n) =>
{
    return n <= 1 ? 1 : n * factorial(n - 1);
};

// 引数と戻り値の型を両方指定
var calculate = double (double x, double y) => x * y + Math.Sqrt(x + y);
```

### 6. 定数式での補間文字列

定数補間文字列をコンパイル時に評価できるようになりました。

```csharp
public class ApiEndpoints
{
    private const string BaseUrl = "https://api.example.com";
    private const string Version = "v1";
    
    // CSharp 10では、定数文字列内で補間が可能
    public const string UsersEndpoint = $"{BaseUrl}/{Version}/users";
    public const string ProductsEndpoint = $"{BaseUrl}/{Version}/products";
}
```

## CSharp 11の新機能

### 1. 必須プロパティ

オブジェクト初期化時に値を設定する必要があるプロパティを定義できます。

```csharp
public class User
{
    // 必須プロパティ
    public required string Username { get; set; }
    public required string Email { get; set; }
    
    // オプションプロパティ
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime? LastLoginDate { get; set; }
}

// 使用例
public class UserService
{
    public void RegisterUser()
    {
        // Username と Email は必須
        var user = new User
        {
            Username = "johndoe",
            Email = "john@example.com",
            FirstName = "John"
            // LastName は省略可能
        };
        
        SaveUser(user);
    }
    
    private void SaveUser(User user)
    {
        // 実装
    }
}
```

### 2. raw 文字列リテラル

改行やエスケープシーケンスを含む文字列をより簡単に記述できます。

```csharp
public class DocumentGenerator
{
    public string GetJsonTemplate()
    {
        // 複数の " が使用可能、インデントも保持される
        string json = """
        {
            "user": {
                "name": "John Doe",
                "email": "john@example.com",
                "roles": [
                    "admin",
                    "editor"
                ]
            }
        }
        """;
        
        return json;
    }
    
    public string GetSqlQuery()
    {
        // SQL クエリを読みやすく記述できる
        string query = """
            SELECT u.Id, u.Username, u.Email, r.Name as RoleName
            FROM Users u
            JOIN UserRoles ur ON u.Id = ur.UserId
            JOIN Roles r ON ur.RoleId = r.Id
            WHERE u.IsActive = 1
            ORDER BY u.Username
            """;
            
        return query;
    }
}
```

### 3. リストパターン

コレクションの内容をパターンマッチングできます。

```csharp
public class ArrayProcessor
{
    public string DescribeArray(int[] numbers)
    {
        return numbers switch
        {
            [] => "空の配列",
            [1, 2, 3] => "1, 2, 3 を含む配列",
            [0, ..] => "0 で始まる配列",
            [.., 100] => "100 で終わる配列",
            [1, _, _, 4] => "4つの要素があり、最初が1、最後が4の配列",
            [>= 1, >= 1, >= 1] => "3つの正の整数を含む配列",
            _ => "その他の配列"
        };
    }
}
```

### 4. 自動デフォルト構造体

構造体のフィールドをコンストラクタで初期化しなくても、デフォルト値で自動的に初期化されるようになりました。

```csharp
public struct CustomerInfo
{
    public string Name;
    public int Age;
    public decimal Balance;
    
    // フィールドを明示的に初期化しなくても、
    // Name は null、Age は 0、Balance は 0.0M で初期化される
    public CustomerInfo(string name)
    {
        Name = name;
        // Age と Balance はデフォルト値が自動的に設定される
    }
}
```

### 5. 静的抽象メンバー

インターフェースで静的メンバーを定義できるようになりました。

```csharp
public interface IFormatter<T>
{
    // 静的抽象メソッド
    static abstract string Format(T value);
    
    // 静的抽象プロパティ
    static abstract string Name { get; }
}

public class IntFormatter : IFormatter<int>
{
    public static string Format(int value)
    {
        return value.ToString("N0");
    }
    
    public static string Name => "整数フォーマッタ";
}

public class Program
{
    public void DisplayExample()
    {
        Console.WriteLine($"フォーマッタ名: {IntFormatter.Name}");
        Console.WriteLine($"フォーマット結果: {IntFormatter.Format(1234567)}");
    }
}
```

### 6. 改良された型変換

数値リテラルとの演算でベースの型を超えてもエラーにならずに変換されるようになりました。

```csharp
public class MathCalculations
{
    public void PerformCalculations()
    {
        int a = int.MaxValue;
        
        // CSharp 11以前ではオーバーフローでエラー
        // CSharp 11ではlongに自動的に拡張される
        var result = a + 1;  // result は long 型
        
        Console.WriteLine($"結果: {result}, 型: {result.GetType().Name}");
    }
}
```

## CSharp 12の新機能

### 1. プライマリコンストラクタ（クラスとストラクト）

クラスやストラクトでもプライマリコンストラクタ構文が使えるようになりました。

```csharp
// CSharp 12のプライマリコンストラクタ
public class Person(string firstName, string lastName)
{
    public string FirstName { get; } = firstName;
    public string LastName { get; } = lastName;
    public string FullName => $"{FirstName} {LastName}";
    
    // 追加のコンストラクタも定義可能
    public Person(string fullName) : this(fullName.Split(' ')[0], fullName.Split(' ')[1])
    {
    }
}

// 使用例
public class PersonService
{
    public Person CreatePerson()
    {
        return new Person("John", "Doe");
    }
}
```

### 2. コレクション式

配列やリストなどのコレクションをより簡潔に初期化できます。

```csharp
public class CollectionExamples
{
    public void DemonstrateCollectionExpressions()
    {
        // 配列
        int[] numbers = [1, 2, 3, 4, 5];
        
        // リスト
        List<string> fruits = ["Apple", "Banana", "Cherry"];
        
        // スプレッド演算子を使用した結合
        string[] colors1 = ["Red", "Green"];
        string[] colors2 = ["Blue", "Yellow"];
        string[] allColors = [..colors1, ..colors2, "Purple"];
        
        // ディクショナリ
        Dictionary<string, int> scores = new()
        {
            ["Alice"] = 95,
            ["Bob"] = 87,
            ["Charlie"] = 92
        };
    }
}
```

### 3. インラインの配列

ローカル関数やラムダ式内で使用する配列を簡潔に宣言できます。

```csharp
public class ArrayProcessor
{
    public int[] ProcessData(int multiplier)
    {
        // インラインの配列
        int[] Process()
        {
            var source = [1, 2, 3, 4, 5];
            var result = new int[source.Length];
            
            for (int i = 0; i < source.Length; i++)
            {
                result[i] = source[i] * multiplier;
            }
            
            return result;
        }
        
        return Process();
    }
}
```

### 4. ref readonly パラメータ

参照渡しだが変更不可のパラメータを宣言できます。

```csharp
public class DataProcessor
{
    public void ProcessLargeData(ref readonly byte[] data)
    {
        // data は参照渡しだが、変更はできない
        var checksum = CalculateChecksum(data);
        Console.WriteLine($"チェックサム: {checksum}");
        
        // data[0] = 0; // これはコンパイルエラー
    }
    
    private int CalculateChecksum(byte[] data)
    {
        int sum = 0;
        foreach (var b in data)
        {
            sum += b;
        }
        return sum;
    }
}
```

### 5. 別名型

既存の型に別名を付けることができます。

```csharp
// 既存の型に別名をつける
using CustomerId = System.Guid;
using UserName = string;
using ConnectionString = string;

public class UserRepository
{
    private readonly ConnectionString connectionString;
    
    public UserRepository(ConnectionString connectionString)
    {
        this.connectionString = connectionString;
    }
    
    public Task<User> GetUserAsync(CustomerId id)
    {
        // 実装
        return Task.FromResult(new User { Id = id });
    }
    
    public Task<bool> IsUserNameAvailableAsync(UserName userName)
    {
        // 実装
        return Task.FromResult(true);
    }
}
```

### 6. 半自動プロパティ

フィールドにバッキングするプロパティを簡潔に記述できます。

```csharp
public class Product
{
    // プライベートフィールド
    private string name;
    
    // 半自動プロパティ（CSharp 12）
    public string Name
    {
        get => name;
        set
        {
            if (string.IsNullOrEmpty(value))
            {
                throw new ArgumentException("製品名を空にすることはできません");
            }
            name = value;
        }
    }
    
    // 完全に自動化されたプロパティ
    public decimal Price { get; set; }
    
    // サンプルメソッド
    public string GetProductInfo()
    {
        return $"製品名: {Name}, 価格: {Price:C}";
    }
}
```

### 7. 拡張された nameof スコープ

`nameof`の対象範囲が拡張され、より多くの場所で使用できるようになりました。

```csharp
public class ValidationHelper
{
    public void ValidateObject<T>(T obj, string propertyName)
    {
        // パラメータの検証
        if (obj == null)
        {
            throw new ArgumentNullException(nameof(obj));
        }
        
        // 属性内でも nameof が使用可能に
        [Required(ErrorMessage = $"{nameof(T)} ID is required")]
        public int Id { get; set; }
    }
}
```

## CSharp 13の新機能

### 1. インターセプタ

メソッド呼び出しを傍受して前処理や後処理を追加できる強力な機能です。

```csharp
using System.Runtime.CompilerServices;

// インターセプタの定義
[InterceptsLocation("Program.cs", line: 24, character: 8)]
public static void LogMethodCall(string message)
{
    Console.WriteLine($"[LOG] {DateTime.Now}: 呼び出し開始 - {message}");
    RealLogMethodCall(message); // 元のメソッドを呼び出し
    Console.WriteLine($"[LOG] {DateTime.Now}: 呼び出し終了 - {message}");
}

// 本来のメソッド
public static void RealLogMethodCall(string message)
{
    Console.WriteLine($"実際の処理: {message}");
}

// 使用例
public class Application
{
    public void Run()
    {
        // このメソッド呼び出しはインターセプタによって傍受される
        LogMethodCall("アプリケーション開始");
    }
}
```

### 2. 拡張されたパターンマッチング

スイッチ式とパターンマッチングにさらなる拡張が追加されました。

```csharp
public class PatternMatchingExamples
{
    public string DescribeShape(object shape)
    {
        return shape switch
        {
            // プロパティパターンの改善
            Circle { Radius: > 10 } c => $"大きな円 (半径: {c.Radius})",
            Rectangle { Width: var w, Height: var h } when w == h => "正方形",
            Rectangle { Width: > 0, Height: > 0 } r => $"長方形 (幅: {r.Width}, 高さ: {r.Height})",
            
            // 型パターンと論理演算子の組み合わせ
            IShape and not (Circle or Rectangle) => "円と長方形以外の図形",
            
            // 深いプロパティパターン
            Shape { Origin: { X: 0, Y: 0 } } => "原点に位置する図形",
            
            _ => "未知の図形"
        };
    }
}

public interface IShape { }
public record Circle(double Radius, Point Origin) : IShape;
public record Rectangle(double Width, double Height, Point Origin) : IShape;
public record Point(double X, double Y);
```

### 3. 構造化された並行処理

並行処理を構造化して管理しやすくする API が追加されました。

```csharp
public class DataProcessor
{
    public async Task ProcessDataConcurrentlyAsync(List<DataItem> items)
    {
        // 並行処理の制限を設定
        var options = new ParallelOptions
        {
            MaxDegreeOfParallelism = Environment.ProcessorCount
        };
        
        // 構造化された並行処理
        using var concurrencyLimiter = new SemaphoreSlim(options.MaxDegreeOfParallelism);
        
        // タスクの作成と実行
        var tasks = items.Select(async item =>
        {
            await concurrencyLimiter.WaitAsync();
            try
            {
                await ProcessItemAsync(item);
            }
            finally
            {
                concurrencyLimiter.Release();
            }
        });
        
        // すべてのタスクの完了を待機
        await Task.WhenAll(tasks);
    }
    
    private async Task ProcessItemAsync(DataItem item)
    {
        await Task.Delay(100); // シミュレーション
        Console.WriteLine($"項目 {item.Id} を処理しました");
    }
}

public class DataItem
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

### 4. ディスカードパターンの拡張

ディスカードパターンがより多くのコンテキストで使用できるようになりました。

```csharp
public class DiscardPatternExamples
{
    public void ProcessTransaction(Transaction transaction)
    {
        // 拡張されたディスカードパターン
        var (amount, type, _) = transaction;  // 日付を無視
        
        Console.WriteLine($"処理: {type} 取引、金額: {amount:C}");
        
        // switch 式でのディスカード
        string status = transaction switch
        {
            (> 1000, "入金", _) => "大口入金",
            (_, "出金", var date) when date.DayOfWeek == DayOfWeek.Sunday => "休日出金",
            (< 0, _, _) => "無効な取引",
            _ => "通常取引"
        };
        
        Console.WriteLine($"ステータス: {status}");
    }
}

public record Transaction(decimal Amount, string Type, DateTime Date);
```

### 5. 部分メソッドの改善

部分メソッドの制限が緩和され、より柔軟に使用できるようになりました。

```csharp
public partial class OrderProcessor
{
    public void ProcessOrder(Order order)
    {
        // 注文の前処理
        PreProcessOrder(order);
        
        // 基本的な処理
        Console.WriteLine($"注文 {order.Id} を処理しています...");
        
        // カスタム処理（実装は別のファイルで定義可能）
        CustomProcessing(order);
        
        // 注文の後処理
        PostProcessOrder(order);
    }
    
    // 実装は必須、戻り値と例外をスローできる
    partial void CustomProcessing(Order order);
    
    // デフォルト実装を持つ部分メソッド
    partial void PreProcessOrder(Order order)
    {
        Console.WriteLine("標準前処理を実行中...");
    }
    
    partial void PostProcessOrder(Order order);
}

// 別のファイルでの実装
public partial class OrderProcessor
{
    partial void CustomProcessing(Order order)
    {
        if (order.Amount > 10000)
        {
            Console.WriteLine("大口注文の特別処理を実行中...");
        }
    }
    
    partial void PostProcessOrder(Order order)
    {
        Console.WriteLine("注文完了の通知を送信中...");
    }
}

public class Order
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
}
```

### 6. 拡張された属性の対象範囲

属性を適用できる対象範囲が拡張され、より細かい制御が可能になりました。

```csharp
// ラムダ式に属性を適用
var calculate = [LogExecutionTime] (int x, int y) => x * y + x / y;

// ローカル関数に属性を適用
[LogExecutionTime]
double CalculateDistance(double x1, double y1, double x2, double y2)
{
    return Math.Sqrt(Math.Pow(x2 - x1, 2) + Math.Pow(y2 - y1, 2));
}

// 型パラメータに属性を適用
public class Repository<[Serializable] T> where T : class
{
    public void Save(T entity)
    {
        // 実装
    }
}

// カスタム属性の例
[AttributeUsage(AttributeTargets.Method | AttributeTargets.Class)]
public class LogExecutionTimeAttribute : Attribute
{
    // 属性の実装
}
```

### 7. 動的コンパイルの改善

コンパイル時の最適化と動的コード生成の機能が向上しました。

```csharp
public class DynamicCodeExample
{
    public void GenerateAndExecuteCode()
    {
        // 動的コンパイルの例
        var sourceCode = @"
            using System;
            
            namespace DynamicCode
            {
                public class Calculator
                {
                    public double Add(double a, double b) => a + b;
                    public double Multiply(double a, double b) => a * b;
                }
            }";
            
        // コンパイルオプションの設定
        var options = new CompilationOptions(
            optimizationLevel: OptimizationLevel.Release,
            enableNullableReferenceTypes: true);
            
        // 動的コンパイルの実行
        // 注: 実際の実装はコンパイラAPIに依存します
        var assembly = CompileSource(sourceCode, options);
        
        // 動的に生成されたコードの実行
        var calculatorType = assembly.GetType("DynamicCode.Calculator");
        var calculator = Activator.CreateInstance(calculatorType);
        var result = calculatorType.GetMethod("Add").Invoke(calculator, new object[] { 5.0, 3.0 });
        
        Console.WriteLine($"動的コードの実行結果: {result}");
    }
    
    // 注: これは簡略化された例です
    private Assembly CompileSource(string source, CompilationOptions options)
    {
        // 実際の実装はここに記述
        return null;
    }
}

public class CompilationOptions
{
    public OptimizationLevel optimizationLevel { get; }
    public bool enableNullableReferenceTypes { get; }
    
    public CompilationOptions(OptimizationLevel optimizationLevel, bool enableNullableReferenceTypes)
    {
        this.optimizationLevel = optimizationLevel;
        this.enableNullableReferenceTypes = enableNullableReferenceTypes;
    }
}

public enum OptimizationLevel
{
    Debug,
    Release
}
```

## まとめ

CSharp 10以降は、コードの読みやすさと書きやすさを向上させる多くの機能が追加されています。これらの新機能を活用することで、より簡潔で表現力豊かなコードを書くことができます。特に以下の機能に注目すると良いでしょう：

1. ファイルスコープの名前空間宣言（コードの階層を減らす）
2. グローバル using ディレクティブ（共通の名前空間を一箇所で管理）
3. raw 文字列リテラル（JSON や SQL などを読みやすく書ける）
4. 必須プロパティ（初期化が必要なプロパティを明示できる）
5. コレクション式（配列やリストをより簡潔に初期化できる）
6. プライマリコンストラクタ（クラスでのコンストラクタ宣言を簡略化）
7. インターセプタ（メソッド呼び出しの前後に処理を追加できる）
8. 構造化された並行処理（並行処理をより管理しやすくする）

これらの機能は、日々のコーディング作業を効率化し、より読みやすいコードベースの維持に役立ちます。継続的にC#の新機能を学び、適切に活用することで、より効率的で堅牢なアプリケーション開発が可能になります。