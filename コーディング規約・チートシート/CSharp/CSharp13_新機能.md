---
作成日: 2025-05-27
tags:
  - CSharp
子文書:
---
# CSharp 13.0 新機能まとめ

CSharp 13.0では、コードの簡潔さ、型安全性、パフォーマンスの向上に焦点を当てた新機能が追加されました。

## 1. パラメータリストのパターンマッチング

メソッドのパラメータリストでパターンマッチングを使用できるようになりました。

```csharp
// パラメータリストでのパターンマッチング
public void ProcessData((string Name, int Age) person)
{
    // パラメータの分解
    var (name, age) = person;
    Console.WriteLine($"{name}さんは{age}歳です");
}

// 複雑なパターンマッチング
public string GetPersonInfo((string Name, int Age, string Department) person) => person switch
{
    ("田中", 30, "開発部") => "田中さんは開発部の30歳です",
    (_, var age, "営業部") when age < 30 => "若手営業マンです",
    (var name, var age, _) => $"{name}さんは{age}歳です",
    _ => "不明な情報です"
};
```

## 2. 拡張メソッドの型パラメータ

拡張メソッドで型パラメータを使用できるようになりました。

```csharp
// 拡張メソッドでの型パラメータ
public static class EnumerableExtensions
{
    public static IEnumerable<TResult> SelectMany<T, TResult>(
        this IEnumerable<T> source,
        Func<T, IEnumerable<TResult>> selector)
    {
        foreach (var item in source)
        {
            foreach (var result in selector(item))
            {
                yield return result;
            }
        }
    }
}

// 使用例
var numbers = new[] { 1, 2, 3 };
var result = numbers.SelectMany(n => Enumerable.Range(1, n));
```

## 3. インターフェースのデフォルト実装の改善

インターフェースのデフォルト実装がより柔軟になりました。

```csharp
public interface ILogger
{
    void Log(string message);
    
    // デフォルト実装を持つメソッド
    void LogError(string message) => Log($"ERROR: {message}");
    void LogWarning(string message) => Log($"WARNING: {message}");
    void LogInfo(string message) => Log($"INFO: {message}");
}

// 実装例
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
    
    // デフォルト実装をオーバーライド
    public void LogError(string message)
    {
        Console.WriteLine($"重大なエラー: {message}");
    }
}
```

## 4. レコード型の機能強化

レコード型の機能が強化され、より柔軟な使用が可能になりました。

```csharp
// レコード型の機能強化
public record Person
{
    public required string Name { get; init; }
    public required int Age { get; init; }
    public string? Email { get; init; }
    
    // カスタムデコンストラクタ
    public void Deconstruct(out string name, out int age)
    {
        name = Name;
        age = Age;
    }
}

// 使用例
var person = new Person { Name = "田中太郎", Age = 30 };
var (name, age) = person;
```

## 5. パターンマッチングの拡張

パターンマッチングの機能がさらに拡張されました。

```csharp
// 拡張されたパターンマッチング
public string GetPersonStatus(Person person) => person switch
{
    { Name: "田中", Age: > 30 } => "田中さんは30歳以上です",
    { Name: var name, Age: < 20 } => $"{name}さんは未成年です",
    { Email: not null } => "メールアドレスが登録されています",
    _ => "その他の情報です"
};

// リストパターンの拡張
public string ProcessNumbers(int[] numbers) => numbers switch
{
    [1, 2, 3, ..] => "1,2,3で始まります",
    [.., 7, 8, 9] => "7,8,9で終わります",
    [1, .., 9] => "1で始まり9で終わります",
    _ => "その他のパターンです"
};
```

## 6. 非同期ストリームの改善

非同期ストリームの機能が改善され、より効率的な使用が可能になりました。

```csharp
// 非同期ストリームの改善
public async IAsyncEnumerable<int> GetNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}

// 使用例
await foreach (var number in GetNumbersAsync())
{
    Console.WriteLine(number);
}
```

## 7. 型推論の強化

型推論の機能が強化され、より正確な型推論が可能になりました。

```csharp
// 型推論の強化
var numbers = new[] { 1, 2, 3, 4, 5 };
var sum = numbers.Sum(); // int型として推論

var mixed = new[] { 1, 2.0, 3.5 }; // double型として推論
var average = mixed.Average(); // double型として推論
```

## 8. パフォーマンス最適化

パフォーマンス最適化のための新機能が追加されました。

```csharp
// パフォーマンス最適化
public class OptimizedList<T>
{
    private T[] _items;
    private int _count;
    
    public OptimizedList(int capacity)
    {
        _items = new T[capacity];
    }
    
    public void Add(T item)
    {
        if (_count == _items.Length)
        {
            Array.Resize(ref _items, _items.Length * 2);
        }
        _items[_count++] = item;
    }
}
```

## まとめ

CSharp 13.0では以下の主要な改善がありました：

- **パラメータリストのパターンマッチング**: メソッドパラメータでの柔軟なパターンマッチング
- **拡張メソッドの型パラメータ**: より柔軟な拡張メソッドの実装
- **インターフェースのデフォルト実装の改善**: より柔軟なインターフェース設計
- **レコード型の機能強化**: より表現力豊かなレコード型の使用
- **パターンマッチングの拡張**: より強力なパターンマッチング機能
- **非同期ストリームの改善**: より効率的な非同期処理
- **型推論の強化**: より正確な型推論
- **パフォーマンス最適化**: より効率的なコード実行

これらの機能により、CSharpのコードはより簡潔で読みやすくなり、型安全性とパフォーマンスも向上しました。特にパラメータリストのパターンマッチングとレコード型の機能強化は、日常的な開発で大きな恩恵をもたらします。 