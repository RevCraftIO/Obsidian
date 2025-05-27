---
作成日: 2025-05-27
tags:
  - CSharp
子文書:
---

# C# 12.0 新機能まとめ

C# 12.0では、コードの簡潔さと型安全性の向上に焦点を当てた新機能が追加されました。

## 1. Primary Constructors

クラスと構造体でプライマリコンストラクタを使用できるようになりました。

```csharp
// クラスのプライマリコンストラクタ
public class Person(string name, int age)
{
    public string Name { get; } = name;
    public int Age { get; } = age;
    
    public string GetInfo() => $"{Name} ({Age}歳)";
}

// 構造体のプライマリコンストラクタ
public readonly struct Point(double x, double y)
{
    public double X { get; } = x;
    public double Y { get; } = y;
    
    public double Distance => Math.Sqrt(X * X + Y * Y);
}

// 使用例
var person = new Person("田中太郎", 30);
var point = new Point(3, 4);
```

## 2. Collection Expressions

コレクションの初期化がより簡潔になりました。

```csharp
// 配列の初期化
int[] numbers = [1, 2, 3, 4, 5];
string[] names = ["田中", "鈴木", "佐藤"];

// リストの初期化
List<int> scores = [85, 92, 78, 95, 88];
List<string> cities = ["東京", "大阪", "名古屋"];

// スパンでの使用
Span<int> values = [10, 20, 30, 40, 50];

// コレクションの結合
int[] combined = [..numbers, ..scores];
```

## 3. Default Lambda Parameters

ラムダ式でデフォルトパラメータを使用できるようになりました。

```csharp
// デフォルトパラメータ付きラムダ式
var add = (int x, int y = 1) => x + y;
var greet = (string name = "ゲスト") => $"こんにちは、{name}さん！";

// 使用例
Console.WriteLine(add(5));      // 6
Console.WriteLine(add(5, 3));   // 8
Console.WriteLine(greet());     // こんにちは、ゲストさん！
Console.WriteLine(greet("田中")); // こんにちは、田中さん！
```

## 4. Alias Any Type

任意の型にエイリアスを付けることができます。

```csharp
// 型エイリアスの定義
using Point = (double X, double Y);
using UserInfo = (string Name, int Age, string Email);

// 使用例
Point p = (3.0, 4.0);
UserInfo user = ("田中太郎", 30, "tanaka@example.com");

// メソッドでの使用
double GetDistance(Point p1, Point p2) =>
    Math.Sqrt(Math.Pow(p2.X - p1.X, 2) + Math.Pow(p2.Y - p1.Y, 2));
```

## 5. Inline Arrays

固定サイズの配列をインラインで定義できます。

```csharp
// インライン配列の定義
[System.Runtime.CompilerServices.InlineArray(5)]
public struct FixedArray<T>
{
    private T _element0;
}

// 使用例
var numbers = new FixedArray<int>();
numbers[0] = 1;
numbers[1] = 2;
numbers[2] = 3;
numbers[3] = 4;
numbers[4] = 5;
```

## 6. Interpolated String Improvements

補間文字列の機能が強化されました。

```csharp
// 複数行の補間文字列
string message = $"""
    ユーザー情報:
    名前: {user.Name}
    年齢: {user.Age}
    メール: {user.Email}
    """;

// 条件付き補間
string status = $"状態: {(isActive ? "有効" : "無効")}";

// 式の補間
string result = $"計算結果: {Math.Sqrt(16)}";
```

## 7. Pattern Matching Improvements

パターンマッチングの機能が強化されました。

```csharp
// リストパターンの拡張
int[] numbers = [1, 2, 3, 4, 5];
string result = numbers switch
{
    [1, 2, 3, ..] => "1,2,3で始まる",
    [.., 4, 5] => "4,5で終わる",
    [1, .., 5] => "1で始まり5で終わる",
    _ => "その他"
};

// プロパティパターンの拡張
public record Person(string Name, int Age, string Department);

string GetPersonInfo(Person person) => person switch
{
    { Name: "田中", Age: > 30 } => "田中さん（30歳以上）",
    { Department: "開発部", Age: < 30 } => "若手開発者",
    { Name: var name, Age: var age } => $"{name}（{age}歳）",
    _ => "不明"
};
```

## 8. Required Properties Improvements

必須プロパティの機能が強化されました。

```csharp
public class Product
{
    public required string Name { get; init; }
    public required decimal Price { get; init; }
    public string? Description { get; init; }
    
    // コンストラクタでの必須プロパティの設定
    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}

// 使用例
var product = new Product("ノートPC", 89800)
{
    Description = "高性能ノートPC"
};
```

## 9. File-scoped Types

ファイルスコープの型定義が可能になりました。

```csharp
// ファイルスコープの型定義
file class Helper
{
    public static string FormatName(string name) => name.Trim();
}

// 同じファイル内でのみ使用可能
public class UserService
{
    public string ProcessName(string name) => Helper.FormatName(name);
}
```

## 10. Generic Type Parameters

ジェネリック型パラメータの機能が強化されました。

```csharp
// ジェネリック型パラメータの制約
public class Repository<T> where T : class, new()
{
    public T Create() => new T();
}

// ジェネリック型パラメータの属性
public class Validator<T> where T : notnull
{
    public bool Validate(T value) => value != null;
}
```

## まとめ

C# 12.0では以下の主要な改善がありました：

- **Primary Constructors**: クラスと構造体での簡潔なコンストラクタ定義
- **Collection Expressions**: コレクション初期化の簡素化
- **Default Lambda Parameters**: ラムダ式でのデフォルトパラメータ
- **Alias Any Type**: 任意の型へのエイリアス
- **Inline Arrays**: 固定サイズ配列の簡潔な定義
- **Pattern Matching Improvements**: パターンマッチングの機能強化

これらの機能により、C#のコードはより簡潔で読みやすくなり、型安全性も向上しました。特にPrimary ConstructorsとCollection Expressionsは日常的な開発で大きな恩恵をもたらします。
