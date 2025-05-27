## 子文書
- [CSharp_新機能](CSharp_新機能.md)
- [[ASP.NET Core Web API]]
- [[WPF]]

## 目次

1. [命名規則](#命名規則)
2. [フォーマットと構文](#フォーマットと構文)
3. [コメント](#コメント)
4. [言語の機能と使用法](#言語の機能と使用法)
5. [例外処理](#例外処理)
6. [並列処理とパフォーマンス](#並列処理とパフォーマンス)
7. [セキュリティ](#セキュリティ)
8. [テスト](#テスト)
9. [バージョン管理](#バージョン管理)
10. [ツールとリソース](#ツールとリソース)

## 命名規則

### 全般

- **Pascal Case**: クラス、メソッド、プロパティ、名前空間、列挙型に使用します。
  - 例: `EmployeeService`, `CalculateTax()`

- **Camel Case**: 変数、パラメータに使用します。
  - 例: `employeeCount`, `firstName`

- **UPPER_CASE**: 定数に使用します。
  - 例: `MAX_SIZE`, `CONNECTION_STRING`

- **privateField**: プライベートフィールドには camel case を使用し、先頭にアンダースコアを付けることは禁止します。
  - 例: `userRepository`, `logger`

### 具体的な命名規則

#### クラス
- 名詞または名詞句を使用します。
- プレフィックスは使用しません（例: `C`や`cls`などは付けない）。
- 例: `Customer`, `Account`, `FileStream`

#### インターフェース
- プレフィックスとして `I` を使用します。
- 例: `IDisposable`, `IRepository`, `INotifyPropertyChanged`

#### メソッド
- 動詞または動詞句を使用します。
- 例: `SaveChanges()`, `GetById()`, `CalculateTotal()`

#### プロパティ
- 名詞または名詞句を使用します。
- 例: `FirstName`, `OrderCount`, `IsValid`

#### イベント
- 動詞の過去形または現在進行形を使用します。
- 例: `Clicked`, `Opening`, `Closing`, `Changed`

#### 列挙型
- 単数形の名詞を使用します。
- 例: `FileMode`, `Color`, `UserType`

#### ブール値
- `Is`, `Has`, `Can`, `Should` などで始めます。
- 例: `IsEnabled`, `HasChildren`, `CanExecute`

### 禁止事項

- ハンガリアン記法は使用しません（例: `strName`, `intCount`）。
- 予約語を識別子として使用しません。

## フォーマットと構文

### インデントとスペース

- インデントには**スペース4つ**を使用します。タブは使用しません。
- 中括弧 `{` は新しい行に配置します（Allman スタイル）。
- メソッドや制御構造の間に空行を挿入してコードを整理します。

```csharp
public class Example
{
    public void Method()
    {
        if (condition)
        {
            // 処理
        }
    }
}
```

### 行の長さと折り返し

- 1行の長さは120文字以内にします。
- パラメータリストが長い場合は、各パラメータを別の行にします。

```csharp
public void LongMethodWithManyParameters(
    int firstParameter,
    string secondParameter,
    bool thirdParameter,
    DateTime fourthParameter)
{
    // 処理
}
```

### 中括弧

- 制御構文に対しては、例え一行でも中括弧を常に使用します。

```csharp
// 良い例1
if (condition) { DoSomething(); }

// 良い例2
if (condition)
{
    DoSomething();
}

// 悪い例
if (condition) DoSomething();
```

### using ディレクティブ

- 名前空間の外側に `using` ディレクティブを配置します。
- アルファベット順に並べます。
- System 名前空間を最初に配置します。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using CompanyName.ProductName.Feature.Component;
```

## コメント

### コメントの目的

- 「なぜ」という理由を説明するためにコメントを使用します。「何を」や「どのように」はコード自体で明確であるべきです。
- 複雑なアルゴリズムまたはビジネスルールを説明するためにコメントを使用します。

### XMLドキュメンテーション

- パブリック API（クラス、メソッド、プロパティ）にはすべて XML コメントを使用します。
- 少なくとも `<summary>` タグを含めます。
- パラメータには `<param>` タグ、戻り値には `<returns>` タグ、例外には `<exception>` タグを使用します。

```csharp
/// <summary>
/// 指定されたIDに対応する顧客情報を取得します。
/// </summary>
/// <param name="id">顧客ID</param>
/// <returns>顧客情報。見つからない場合は null。</returns>
/// <exception cref="ArgumentException">IDが無効な場合にスローされます。</exception>
public Customer GetCustomerById(int id)
{
    // 実装
}
```

### コードコメント

- コードコメントは `//` を使用し、コメントの前に1つのスペースを入れます。
- TODO コメントには標準形式を使用します: `// TODO: 説明`

## 言語の機能と使用法

### 型の使用

- 可能な限り var キーワードを使用します（特に型が明らかな場合）。
- ジェネリックコレクションを優先します（`List<T>`, `Dictionary<TKey, TValue>` など）。
- 値型と参照型の違いを理解して使用します。

```csharp
// 良い例
var customers = new List<Customer>();

// 型が明確でない場合は明示的に型を指定
Dictionary<string, List<Order>> customerOrders = GetOrders();
```

### LINQ

- 可読性を重視します。複雑なクエリは複数行に分割します。
- メソッド構文とクエリ構文を状況に応じて使い分けます。

```csharp
// メソッド構文の例
var activeCustomers = customers
    .Where(c => c.IsActive)
    .OrderBy(c => c.LastName)
    .ToList();

// クエリ構文の例 (複雑なクエリに適している場合)
var results = from c in customers
              join o in orders on c.Id equals o.CustomerId
              where c.IsActive && o.OrderDate > DateTime.Today.AddDays(-30)
              select new { Customer = c, Order = o };
```

### 非同期プログラミング

- 非同期メソッドには 'Async' サフィックスを付けます。
- 非同期メソッドは必ず `Task` または `Task<T>` を返します。
- `async void` は避け、イベントハンドラ以外では使用しません。
- `ConfigureAwait(false)` を適切に使用します。

```csharp
public async Task<Customer> GetCustomerByIdAsync(int id)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync().ConfigureAwait(false);
        // 実装
    }
}
```

### null 処理

- C# 8.0以降では null 許容参照型を使用します。
- null チェックには null 条件演算子 (`?.`) と null 合体演算子 (`??`) を活用します。
- パラメータのnullチェックは `ArgumentNullException` を使用します。

```csharp
public void ProcessCustomer(Customer customer)
{
    if (customer == null)
    {
        throw new ArgumentNullException(nameof(customer));
    }
    
    // または
    ArgumentNullException.ThrowIfNull(customer);
    
    // 以降の処理
}
```

### ディスポーザブルオブジェクト

- `IDisposable` を実装するオブジェクトは `using` ステートメントで囲みます。
- C# 8.0以降では `using` 宣言を使用できます。

```csharp
// 従来の using ステートメント
using (var file = File.OpenRead(path))
{
    // ファイル操作
}

// C# 8.0 以降の using 宣言
using var file = File.OpenRead(path);
// ファイル操作
// スコープ終了時に自動的に Dispose される
```

## 例外処理

### 基本原則

- 例外は例外的な状況のみに使用し、通常のフロー制御には使用しません。
- 特定の例外をキャッチし、一般的な `Exception` のキャッチは避けます。
- 例外をキャッチする場合は適切に処理するか、意味のある情報を追加して再スローします。

```csharp
try
{
    // 例外が発生する可能性のある処理
}
catch (SqlException ex) when (ex.Number == 1205)
{
    // デッドロックの特定処理
    logger.LogWarning(ex, "データベースデッドロックが発生しました。リトライします。");
    return await RetryOperation();
}
catch (SqlException ex)
{
    // その他のSQL例外の処理
    logger.LogError(ex, "データベース操作に失敗しました。");
    throw new DatabaseOperationException("データ取得に失敗しました", ex);
}
```

### カスタム例外

- カスタム例外はシステム固有の例外条件のために作成します。
- 名前は必ず `Exception` で終わるようにします。
- 常にシリアライズ可能に設計し、少なくとも3つのコンストラクタを実装します。

```csharp
[Serializable]
public class ProductNotFoundException : Exception
{
    public ProductNotFoundException() : base() { }
    
    public ProductNotFoundException(string message) : base(message) { }
    
    public ProductNotFoundException(string message, Exception innerException) 
        : base(message, innerException) { }
    
    protected ProductNotFoundException(SerializationInfo info, StreamingContext context) 
        : base(info, context) { }
}
```

## 並列処理とパフォーマンス

### 並列処理

- 適切な場所で `Parallel`, `Task.Run`, `async/await` を使用します。
- スレッドセーフな実装に注意します。
- デッドロックを避けるためにリソースの獲得順序を一貫させます。

```csharp
// 並列処理の例
var items = Enumerable.Range(1, 1000).ToList();
var result = new ConcurrentBag<int>();

Parallel.ForEach(items, item =>
{
    var processed = ProcessItem(item);
    result.Add(processed);
});
```

### パフォーマンス最適化

- プリマチュアな最適化を避けます。
- 最適化の前に計測を行います。
- メモリ使用量に注意し、大きなオブジェクトの作成と破棄を最小限にします。
- 文字列連結には `StringBuilder` を使用します。

```csharp
// 大量の文字列連結はStringBuilderを使用
var sb = new StringBuilder();
foreach (var item in items)
{
    sb.AppendLine(item.ToString());
}
string result = sb.ToString();
```

## セキュリティ

### データ保護

- 機密データは暗号化して保存します。
- 機密情報をログに記録しません。
- パスワードやAPIキーなどの秘密情報はハードコーディングしません。

### 入力検証

- すべてのユーザー入力を検証し、サニタイズします。
- SQLインジェクションを防ぐためにパラメータ化クエリを使用します。
- クロスサイトスクリプティング(XSS)を防止するために出力をエンコードします。

```csharp
// パラメータ化クエリの例
using (var command = new SqlCommand("SELECT * FROM Users WHERE Username = @Username", connection))
{
    command.Parameters.AddWithValue("@Username", username);
    // 実行
}
```

### OWASP

- OWASPトップ10の脆弱性に対する対策を実装します。
- 認証と認可のロジックを慎重に設計・実装します。

## テスト

### 単体テスト

- すべての公開APIには単体テストを作成します。
- テストはARRANGE-ACT-ASSERT パターンで構成します。
- テストケース名は「テスト対象_条件_期待される結果」の形式にします。

```csharp
[Fact]
public void CalculateTax_WhenPriceIsNegative_ThrowsArgumentException()
{
    // Arrange
    var calculator = new TaxCalculator();
    decimal price = -100;
    
    // Act & Assert
    Assert.Throws<ArgumentException>(() => calculator.CalculateTax(price));
}
```

### モック

- インターフェースを使用して依存関係をモック可能にします。
- テストではモックライブラリ（Moq, NSubstitute など）を使用します。

```csharp
[Fact]
public async Task GetCustomerById_WhenCustomerExists_ReturnsCustomer()
{
    // Arrange
    var expectedCustomer = new Customer { Id = 1, Name = "Test Customer" };
    var mockRepository = new Mock<ICustomerRepository>();
    mockRepository.Setup(repo => repo.GetByIdAsync(1))
                  .ReturnsAsync(expectedCustomer);
    
    var service = new CustomerService(mockRepository.Object);
    
    // Act
    var result = await service.GetCustomerByIdAsync(1);
    
    // Assert
    Assert.Equal(expectedCustomer.Id, result.Id);
    Assert.Equal(expectedCustomer.Name, result.Name);
}
```

## バージョン管理

### コミットメッセージ

- コミットメッセージは明確で簡潔に書きます。
- 変更の理由と内容を含めます。
- コミットメッセージの最初の行は50文字以内、必要に応じて詳細を追加します。

```
機能: ユーザー認証機能の追加

- ログイン画面とロジックの実装
- パスワードハッシュ化処理の追加
- ユーザーセッション管理の実装
```

### ブランチ戦略

- 機能ブランチ、リリースブランチ、ホットフィックスブランチを適切に使い分けます。
- プルリクエストを利用してコードレビューを実施します。
- マージ前に自動テストを実行します。

## ツールとリソース

### 推奨ツール

- **IDE**: Visual Studio または Visual Studio Code
- **コード分析**: StyleCop, FxCop, SonarQube
- **テストフレームワーク**: xUnit, NUnit, MSTest
- **モックフレームワーク**: Moq, NSubstitute
- **ビルド自動化**: Azure DevOps, GitHub Actions, Jenkins

### コード分析の設定

- EditorConfig ファイルを使用してコードスタイルを統一します。
- コードアナライザーを有効にして警告をエラーとして扱います。

```editorconfig
# .editorconfig の例
root = true

[*.cs]
indent_style = space
indent_size = 4
end_of_line = crlf
insert_final_newline = true
charset = utf-8-bom

# C# 固有の設定
csharp_new_line_before_open_brace = all
csharp_prefer_braces = true:warning
dotnet_sort_system_directives_first = true
```

### 継続的な学習

- 新しいC#バージョンとベストプラクティスを継続的に学びます。
- コードレビューを通じて知識を共有します。
- 定期的にコーディング規約を見直し、必要に応じて更新します。
