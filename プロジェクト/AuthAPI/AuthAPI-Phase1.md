
---

## フェーズ１：ASP.NET Core Web APIの基本とIdentityの導入

---

### ASP.NET Core Web APIプロジェクトの作成
- プロジェクトテンプレートの一覧から「ASP.NET Core Web API」を選択し、「次へ」をクリックします。
- プロジェクト名として `AuthAPI` などの任意の名前で作成します。
- ターゲットフレームワークは最新の推奨されるもの(.NET 9)を選択します。
- 認証の種類は「認証なし」を選択します。

---

### クラスライブラリの作成
- このクラスライブラリはDTOなど、クライアントと共通で使用するクラスを定義します。
- プロジェクト名として `AuthAPI.Shared` などの任意の名前で作成します。

---

### データクラスの実装
#### AppUserクラスの実装
- `AuthAPI` > `Models` > `Entities` >   `AppUser.cs`
- `AppUser` クラスは `IdentityUser<Guid>` を継承します。

| プロパティ名 | データ型 | 説明 |
|-------------|---------|------|
| LastName | string | 姓 |
| FirstName | string | 名 |

#### UserInfoクラスの実装
- `AuthAPI.Shared` > `Models` > `Entities` > `UserInfo.cs`
- この `UserInfo` クラスはJSONファイルにユーザー情報を永続化するために使用されます。
- 各プロパティには `Description` 属性と `Display` 属性を付与します。

| プロパティ名 | データ型 | 説明 |
|-------------|---------|------|
| UserId | Guid | ユーザーID |
| UserName | string | ユーザー名(ログインID) |
| NormalizedUserName | string | 正規化されたユーザー名(ログインID) |
| LastName | string | 姓 |
| FirstName | string | 名 |
| PasswordHash | string | パスワードハッシュ |
| SecurityStamp | string | セキュリティスタンプ (パスワード変更や重要なセキュリティ変更時に更新)  |`SecurityStamp` (セキュリティ関連トークンの無効化用) |
| ConcurrencyStamp | string | 同時実行スタンプ (オプティミスティック同時実行制御用) | `ConcurrencyStamp` |
| LockoutEnd | DateTimeOffset? | アカウントロックアウトの終了日時 (UTC)  | `LockoutEnd` (null許容) |
| LockoutEnabled | bool | このアカウントでロックアウト機能が有効か | `LockoutEnabled` |
| AccessFailedCount | int | 連続したログイン失敗回数 | `AccessFailedCount` |

---

### AppSettingsクラスの実装
- AuthAPI` > `Models` > `Settings` > `AppSettings.cs`
- `appsettings.json` に保存した情報を取得するために `AppSettings` クラスを作成します。
- アプリケーション起動時に、`appsettings.json`の内容を`AppSettings`クラスにマッピングします。
- 利用時はコンストラクタで `IOptions<AppSettings>` を受け取ることで設定を使用できます。

#### AppSettingsクラスのプロパティ一覧

| プロパティ名 | データ型 | 説明 |
|-------------|---------|------|
| UsersFilePath | string | ユーザー情報を保存する `users.json` のファイルパス |

---

### UserStoreの実装
- `AuthAPI` > `Stores` > `UserStore.cs`
- `UserStore` クラスに、`IUserStore<AppUser>` と `IUserPasswordStore<AppUser>` インターフェースを実装します。
- `UserStore` クラスには `IOptions<AppSettings>` と `ILogger<UserStore>` の依存性注入をします。
- JSONファイルの存在を確認し、存在しない場合は空のJSON配列で初期化するメソッドを作成します。
- `SemaphoreSlim` を使用して排他制御を行います。
- `UserStore` クラスに、JSONファイルからユーザーデータを読み込み、`List<UserData>` として保持するメソッドを作成します。

#### UserServiceクラスのプライベートメソッド一覧
- `private void LoadUsers()`
- `private void SaveUsers()`

---

### JSONファイルへの初期データ書き込み
- アプリケーション起動時に、 `Admin` ユーザーが存在するか確認します。
- `Admin` ユーザーが存在しない場合、 `UserName` を `Admin` 、パスワードを `Administrator` にして `AppUser` を作成し、登録します。



