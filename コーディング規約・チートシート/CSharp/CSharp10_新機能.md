---
作成日: 2025-05-27
tags:
  - CSharp
子文書:
---
# C# 10.0 新機能まとめ

C# 10.0では生産性の向上とコードの簡素化に焦点を当てた多くの新機能が追加されました。

## 1. Record Structs

値型でもRecordの恩恵を受けることができるようになりました。

### 基本的なRecord Struct

```csharp
// 読み取り専用レコード構造体
public readonly record struct Point(double X, double Y)
{
    public double Distance => Math.Sqrt(X * X + Y * Y);
}

// 可変レコード構造体
public record struct MutablePoint(double X, double Y)
{
    public readonly double Distance => Math.Sqrt(X * X + Y * Y);
}

// 使用例
var point1 = new Point(3, 4);
var point2 = new Point(3, 4);

Console.WriteLine(point1 == point2);        // True
Console.WriteLine(point1.Distance);         // 5
Console.WriteLine(point1);                  // Point { X = 3, Y = 4 }

// with式での使用
var newPoint = point1 with { X = 5 };
Console.WriteLine(newPoint);                // Point { X = 5, Y = 4 }
```

### より複雑なRecord Struct

```csharp
public record struct Employee(int Id, string Name, decimal Salary)
{
    public string Department { get; init; } = "";
    
    public Employee GiveRaise(decimal amount) => this with { Salary = Salary + amount };
    
    public readonly string GetSummary() => $"{Name} (ID: {Id}) - ${Salary:N0}";
}

// 使用例
var employee = new Employee(1, "田中太郎", 50000) { Department = "開発部" };
var promoted = employee.GiveRaise(10000);

Console.WriteLine(employee.GetSummary());   // 田中太郎 (ID: 1) - $50,000
Console.WriteLine(promoted.GetSummary());   // 田中太郎 (ID: 1) - $60,000
```

## 2. Global Using Directives

プロジェクト全体で使用するusingディレクティブを一箇所で管理できます。

```csharp
// GlobalUsings.cs または任意のファイル
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
global using Microsoft.Extensions.Logging;

// 以降、全てのファイルで上記の名前空間が利用可能
// Program.cs
Console.WriteLine("Hello World!"); // System.Consoleが不要

var numbers = new List<int> { 1, 2, 3, 4, 5 }; // System.Collections.Genericが不要
var evenNumbers = numbers.Where(x => x % 2 == 0); // System.Linqが不要
```

### エイリアスを使ったGlobal Using

```csharp
// グローバルエイリアス
global using Json = System.Text.Json;
global using Http = System.Net.Http;

// 使用例（任意のファイルで）
var options = new Json.JsonSerializerOptions();
var client = new Http.HttpClient();
```

## 3. File-scoped Namespace Declarations

ファイルスコープの名前空間宣言により、インデントを削減できます。

```csharp
// 従来の方法
namespace MyApp.Services
{
    public class UserService
    {
        public void CreateUser(string name)
        {
            // 実装
        }
    }
}

// C# 10の新しい方法
namespace MyApp.Services;

public class UserService
{
    public void CreateUser(string name)
    {
        // 実装
    }
}

// より複雑な例
namespace MyApp.Data.Repositories;

using Microsoft.EntityFrameworkCore;

public class UserRepository
{
    private readonly ApplicationDbContext _context;
    
    public UserRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
}
```

## 4. Constant Interpolated Strings

補間文字列が定数として使用できるようになりました。

```csharp
// 基本的な使用例
const string GreetingTemplate = $"Hello, World!";
const string VersionInfo = $"Version: {1}.{0}.{0}";

// より実用的な例
public class Constants
{
    public const string AppName = "MyApplication";
    public const string Version = "2.1.0";
    public const string FullTitle = $"{AppName} v{Version}";
    
    // 設定値での使用
    public const string DefaultConnectionString = $"Server=localhost;Database={AppName}DB;";
    
    // ログメッセージテンプレート
    public const string StartupMessage = $"{AppName} is starting up...";
    public const string ShutdownMessage = $"{AppName} is shutting down...";
}
```

## 5. Extended Property Patterns

プロパティパターンがネストされたプロパティをサポートするようになりました。

```csharp
public record Address(string Street, string City, string Country);
public record Person(string Name, Address Address, int Age);

// ネストされたプロパティパターン
string GetLocationInfo(Person person) => person switch
{
    { Address.Country: "Japan", Address.City: "Tokyo" } => "東京在住",
    { Address.Country: "Japan", Address.City: "Osaka" } => "大阪在住",
    { Address.Country: "Japan" } => "日本在住",
    { Address.Country: "USA", Address.City: var city } => $"アメリカの{city}在住",
    _ => "その他の地域在住"
};

// より複雑な例
public record Company(string Name, Address Headquarters);
public record Employee(string Name, Company Company, decimal Salary);

decimal GetTaxRate(Employee employee) => employee switch
{
    { Company.Headquarters.Country: "Japan", Salary: > 10000000 } => 0.45m,
    { Company.Headquarters.Country: "Japan", Salary: > 5000000 } => 0.30m,
    { Company.Headquarters.Country: "Japan" } => 0.20m,
    { Company.Headquarters.Country: "USA", Company.Headquarters.City: "New York" } => 0.35m,
    _ => 0.25m
};

// 使用例
var person = new Person("田中太郎", new Address("1-1-1", "Tokyo", "Japan"), 30);
Console.WriteLine(GetLocationInfo(person)); // 東京在住
```

## 6. Lambda Improvements

ラムダ式の型推論とnatural type が改善されました。

### Natural Type for Lambda Expressions

```csharp
// 型推論されたラムダ式
var parse = (string s) => int.Parse(s);
var choose = (bool b) => b ? 1 : "two"; // object型として推論

// デリゲート型の推論
Func<string, int> parser = s => int.Parse(s);
Action<string> printer = s => Console.WriteLine(s);

// より複雑な例
var operations = new[]
{
    (string name, Func<int, int, int> op) => (name, op)
}.Select(x => new { x.name, x.op });

// ジェネリックメソッドでの使用
T Process<T>(Func<T> factory) => factory();

var result = Process(() => DateTime.Now); // DateTime型として推論
```

### Lambda式での属性サポート

```csharp
// ラムダ式に属性を適用
var validator = [return: NotNull] (string input) => 
    string.IsNullOrEmpty(input) ? "default" : input;

// パラメータに属性を適用
var processor = ([NotNull] string input, [Range(0, 100)] int value) => 
    $"{input}: {value}";

// より実用的な例
public class ValidationExample
{
    public static Func<string, bool> EmailValidator = 
        [return: NotNull] ([NotNull] string email) =>
        {
            return email.Contains("@") && email.Contains(".");
        };
}
```

## 7. Caller Argument Expression

呼び出し元の引数式を文字列として取得できます。

```csharp
using System.Runtime.CompilerServices;

public static class Assert
{
    public static void IsTrue(bool condition, 
        [CallerArgumentExpression("condition")] string? message = null)
    {
        if (!condition)
        {
            throw new ArgumentException($"Assertion failed: {message}");
        }
    }
}

// 使用例
int x = 5;
int y = 10;

Assert.IsTrue(x > 0);           // Assertion failed: x > 0
Assert.IsTrue(x < y);           // 成功
Assert.IsTrue(x > y);           // Assertion failed: x > y

// より高度な例
public static class Logger
{
    public static void LogValue<T>(T value, 
        [CallerArgumentExpression("value")] string? expression = null)
    {
        Console.WriteLine($"{expression} = {value}");
    }
}

// 使用例
var name = "田中太郎";
var age = 30;
Logger.LogValue(name);          // name = 田中太郎
Logger.LogValue(age * 2);       // age * 2 = 60
Logger.LogValue(DateTime.Now);  // DateTime.Now = 2023/12/25 10:30:00
```

## 8. Generic Attributes

ジェネリック属性を定義できるようになりました。

```csharp
// ジェネリック属性の定義
public class GenericAttribute<T> : Attribute
{
    public T Value { get; }
    
    public GenericAttribute(T value)
    {
        Value = value;
    }
}

// 使用例
[Generic<string>("Hello World")]
public class StringExample { }

[Generic<int>(42)]
public class IntExample { }

[Generic<Type>(typeof(DateTime))]
public class TypeExample { }

// より実用的な例
public class DefaultValueAttribute<T> : Attribute
{
    public T DefaultValue { get; }
    
    public DefaultValueAttribute(T defaultValue)
    {
        DefaultValue = defaultValue;
    }
}

public class ConfigurationSettings
{
    [DefaultValue<int>(30)]
    public int TimeoutSeconds { get; set; }
    
    [DefaultValue<string>("localhost")]
    public string ServerName { get; set; }
    
    [DefaultValue<bool>(true)]
    public bool EnableLogging { get; set; }
}
```

## 9. Sealed ToString() in Records

Recordのvirtual ToString()をsealedにできます。

```csharp
// 基底レコード
public record Person(string Name)
{
    // ToString()をsealedにして派生クラスでのオーバーライドを防ぐ
    public sealed override string ToString() => $"Person: {Name}";
}

// 派生レコード
public record Employee(string Name, string Department) : Person(Name)
{
    // ToString()をオーバーライドできない（コンパイルエラー）
    // public override string ToString() => $"Employee: {Name} in {Department}";
}

// 別の例
public record Vehicle(string Brand, string Model)
{
    public sealed override string ToString() => $"{Brand} {Model}";
}

public record Car(string Brand, string Model, int Doors) : Vehicle(Brand, Model)
{
    // ToStringは基底クラスのものがそのまま使用される
}

// 使用例
var car = new Car("Toyota", "Prius", 4);
Console.WriteLine(car); // Toyota Prius
```

## 10. Assignment and Declaration in Same Deconstruction

分解代入で宣言と代入を同時に行えます。

```csharp
// タプルの分解
(int x, int y) = (1, 2);
(var a, var b) = (3, 4);

// 既存の変数と新しい変数を混在
int existingX = 100;
(existingX, int newY) = GetCoordinates(); // existingXに代入、newYを宣言

// レコードの分解
public record Point(int X, int Y)
{
    public void Deconstruct(out int x, out int y) => (x, y) = (X, Y);
}

var point = new Point(10, 20);
int existingValue = 5;
(existingValue, var newValue) = point; // existingValueに10を代入、newValueを宣言して20を設定

// より実用的な例
public record Person(string FirstName, string LastName, int Age);

var person = new Person("太郎", "田中", 30);
string currentLastName = "既存の苗字";
(var firstName, currentLastName, var age) = person; // 混在した分解代入

Console.WriteLine($"{firstName} {currentLastName} ({age}歳)");
```