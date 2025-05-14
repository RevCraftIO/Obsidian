# IdentityRole クラス

## 定義

`Microsoft.AspNetCore.Identity` 名前空間に属し、`IdentityRole<TKey>` クラスを継承しています。`TKey` は `string` 型です。

```csharp
public class IdentityRole : Microsoft.AspNetCore.Identity.IdentityRole<string>
```

## コンストラクター

| コンストラクター | 説明 |
| --- | --- |
| `IdentityRole()` | `IdentityRole` の新しいインスタンスを初期化します。 |
| `IdentityRole(string roleName)` | `IdentityRole` の新しいインスタンスを初期化し、ロール名を設定します。 |

## プロパティ

| プロパティ | 型 | 説明 |
| --- | --- | --- |
| `ConcurrencyStamp` | `string` | ロールがストアに永続化されるたびに変更する必要があるランダムな値を取得または設定します。 |
| `Id` | `string` | このロールの主キーを取得または設定します。 |
| `Name` | `string` | このロールの名前を取得または設定します。 |
| `NormalizedName` | `string` | このロールの正規化された名前を取得または設定します。 |

## メソッド

| メソッド | 説明 |
| --- | --- |
| `ToString()` | このロールの名前を返します。 |

## ドキュメント

詳細については、[Microsoft のドキュメント](https://learn.microsoft.com/ja-jp/dotnet/api/microsoft.aspnetcore.identity.identityrole?view=aspnetcore-9.0) を参照してください。

## 使用予定のプロパティ
- `Id`
- `Name`
- `NormalizedName`
- `ConcurrencyStamp`
