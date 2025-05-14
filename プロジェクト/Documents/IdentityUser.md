# IdentityUser クラス

## 定義

`Microsoft.AspNetCore.Identity` 名前空間に属し、`IdentityUser<TKey>` クラスを継承しています。`TKey` は `string` 型です。

```csharp
public class IdentityUser : Microsoft.AspNetCore.Identity.IdentityUser<string>
```

## コンストラクター

| コンストラクター | 説明 |
| --- | --- |
| `IdentityUser()` | `IdentityUser` の新しいインスタンスを初期化します。 |
| `IdentityUser(string userName)` | `IdentityUser` の新しいインスタンスを初期化し、ユーザー名を設定します。 |

## プロパティ

| プロパティ | 型 | 説明 |
| --- | --- | --- |
| `AccessFailedCount` | `int` | 現在のユーザーに対して失敗したログイン試行回数を取得または設定します。 |
| `ConcurrencyStamp` | `string` | ユーザーがストアに永続化されるたびに変更する必要があるランダムな値を取得または設定します。 |
| `Email` | `string` | このユーザーのメール アドレスを取得または設定します。 |
| `EmailConfirmed` | `bool` | ユーザーがメール アドレスを確認したかどうかを示すフラグを取得または設定します。 |
| `Id` | `string` | このユーザーの主キーを取得または設定します。 |
| `LockoutEnabled` | `bool` | ユーザーをロックアウトできるかどうかを示すフラグを取得または設定します。 |
| `LockoutEnd` | `DateTimeOffset?` | ユーザー ロックアウトが終了した日時 (UTC) を取得または設定します。 |
| `NormalizedEmail` | `string` | このユーザーの正規化された電子メール アドレスを取得または設定します。 |
| `NormalizedUserName` | `string` | このユーザーの正規化されたユーザー名を取得または設定します。 |
| `PasswordHash` | `string` | このユーザーのパスワードのソルト化されたハッシュ表現を取得または設定します。 |
| `PhoneNumber` | `string` | ユーザーの電話番号を取得または設定します。 |
| `PhoneNumberConfirmed` | `bool` | ユーザーが電話番号を確認したかどうかを示すフラグを取得または設定します。 |
| `SecurityStamp` | `string` | ユーザーの資格情報が変更されるたびに変更する必要があるランダムな値を取得または設定します (例: パスワードの変更、ログインの削除)。 |
| `TwoFactorEnabled` | `bool` | このユーザーに対して 2 要素認証が有効になっているかどうかを示すフラグを取得または設定します。 |
| `UserName` | `string` | このユーザーのユーザー名を取得または設定します。 |

## メソッド

| メソッド | 説明 |
| --- | --- |
| `ToString()` | このユーザーのユーザー名を返します。 |

## ドキュメント

詳細については、[Microsoft のドキュメント](https://learn.microsoft.com/ja-jp/dotnet/api/microsoft.aspnetcore.identity.identityuser?view=aspnetcore-9.0) を参照してください。

## 使用予定のプロパティ
- `Id`
- `UserName`
- `PasswordHash`
- `AccessFailedCount`
- `SecurityStamp`
- 

## 追加予定のプロパティ
- `LoginId`
- `UserNameKana`
- `UserNameRoman`
