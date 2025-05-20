# AuthAPI 実装手順書

## 目次
1. [フェーズ１：ASP.NET Core Web APIの基本とIdentityの導入](#フェーズ１aspnet-core-web-apiの基本とidentityの導入)
2. [フェーズ２：Web APIでのユーザー登録、認証、ユーザー情報取得](#フェーズ２web-apiでのユーザー登録認証ユーザー情報取得)

---

## フェーズ１：ASP.NET Core Web APIの基本とIdentityの導入

### ASP.NET Core Web APIプロジェクトの作成
- プロジェクトテンプレートの一覧から「ASP.NET Core Web API」を選択し、「次へ」をクリックします。
- プロジェクト名として `AuthApi` などの任意の名前で作成します。
- ターゲットフレームワークは最新の推奨されるもの(.NET 9)を選択します。
- 認証の種類は「認証なし」を選択します。

### クラスライブラリの作成
- このクラスライブラリはDTOなど、クライアントと共通で使用するクラスを定義します。
- プロジェクト名として `AuthApi.Shared` などの任意の名前で作成します。

### データクラスの実装

#### AppUserクラスの実装
- `AuthApi` > `Models` > `Entities` >   `AppUser.cs`
- `AppUser` クラスは `IdentityUser<Guid>` を継承します。

| プロパティ名 | データ型 | 説明 |
|-------------|---------|------|
| LastName | string | 姓 |
| FirstName | string | 名 |

#### UserInfoクラスの実装
- `AuthApi.Shared` > `Models` > `Entities` > `UserInfo.cs`
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

### AppSettingsクラスの実装
- AuthApi` > `Models` > `Settings` > `AppSettings.cs`
- `appsettings.json` に保存した情報を取得するために `AppSettings` クラスを作成します。
- アプリケーション起動時に、`appsettings.json`の内容を`AppSettings`クラスにマッピングします。
- 利用時はコンストラクタで `IOptions<AppSettings>` を受け取ることで設定を使用できます。

#### AppSettingsクラスのプロパティ一覧

| プロパティ名 | データ型 | 説明 |
|-------------|---------|------|
| UsersFilePath | string | ユーザー情報を保存する `users.json` のファイルパス |

### UserStoreの実装
- `AuthApi` > `Stores` > `UserStore.cs`
- `UserStore` クラスに、`IUserStore<AppUser>` と `IUserPasswordStore<AppUser>` インターフェースを実装します。
- `UserStore` クラスには `IOptions<AppSettings>` と `ILogger<UserStore>` の依存性注入をします。
- JSONファイルの存在を確認し、存在しない場合は空のJSON配列で初期化するメソッドを作成します。
- `lock` を使用して排他制御を行います。
- `UserStore` クラスに、JSONファイルからユーザーデータを読み込み、`List<UserData>` として保持するメソッドを作成します。

#### UserServiceクラスのプライベートメソッド一覧
- `private void LoadUsers()`
- `private void SaveUsers()`

### JSONファイルへの初期データ書き込み
- アプリケーション起動時に、 `Admin` ユーザーが存在するか確認します。
- `Admin` ユーザーが存在しない場合、 `UserName` を `Admin` 、パスワードを `Administrator` にして `AppUser` を作成し、登録します。

---

## フェーズ２：Web APIでのユーザー登録、認証、ユーザー情報取得

### データ転送オブジェクト設計

#### 認証関連DTO
| ファイル名 | 用途 | 主要プロパティ |
|-----------|------|---------------|
| `LoginRequestDto` | ログイン認証 | UserName, Password, RememberMe |
| `AuthResponseDto` | 認証結果返却 | Token, Expires, UserInfo |

#### ユーザー操作DTO
| ファイル名 | 用途 | 主要プロパティ |
|-----------|------|---------------|
| `RegisterRequestDto` | 新規ユーザー登録 | UserName, Password, LastName, FirstName |
| `UpdateLoginIdRequestDto` | ログインID変更 | CurrentUserName, NewUserName |
| `UpdatePasswordRequestDto` | パスワード変更 | CurrentPassword, NewPassword |
| `UpdateDisplayNameRequestDto` | 表示名変更 | LastName, FirstName |

### APIエンドポイント設計

#### 認証コントローラー (`AuthController`)
|メソッド|パス|認証要否|機能|
|---|---|---|---|
|POST|`/api/auth/login`|❌|ログイン（認証用 Cookie を発行）|
|POST|`/api/auth/logout`|✔️|ログアウト（認証 Cookie のクリア）|
|GET|`/api/auth/me`|✔️|自身の認証情報確認（ユーザー名返却など）|

#### ユーザーコントローラー (`UserController`)
|メソッド|パス|認証要否|機能|
|---|---|---|---|
|POST|`/api/user/register`|✔️|新規ユーザー登録|
|GET|`/api/user`|✔️|全ユーザー一覧の取得|
|GET|`/api/user/{userId}`|✔️|指定ユーザー（ID）情報の取得|
|GET|`/api/user?username={name}`|✔️|指定ユーザー（ログインID）情報の取得|
|GET|`/api/user?displayName={name}`|✔️|指定ユーザー（表示名）情報の取得|
|PATCH|`/api/user/{userId}`|✔️|指定ユーザー情報の部分更新（ログインID／表示名／パスワードなどをリクエストボディで指定）|
|PATCH|`/api/user/me`|✔️|自身のユーザー情報の部分更新|
|DELETE|`/api/user/{userId}`|✔️|指定ユーザーの削除|

### 認証フロー設計

1. **ログイン処理**
   - ユーザー認証状態の有効期間は480分間
   - 連続5回失敗でアカウントロックアウト（10分間）
   - 5回ロックアウトで永久ロックアウト

2. **パスワードポリシー**(開発段階)
   - 最小長4文字

3. **セッション管理**
   - クッキーにHttpOnlyとSecureフラグを設定
   - 同一ユーザーの複数端末ログインを許可

### エラーハンドリング方針

| エラー種別 | HTTPステータス | 対応方針 |
|-----------|---------------|----------|
| 認証失敗 | 401 Unauthorized | 失敗回数をカウントアップ |
| 権限不足 | 403 Forbidden | 管理者権限が必要な操作で発生 |
| バリデーションエラー | 400 Bad Request | 入力値不備の詳細を返却 |
| 競合状態 | 409 Conflict | 同時更新検出時に発生 |
| アカウントロック | 423 Locked | ロックアウト期間を明示 |

### 監査ログ要件

以下の操作をJSONファイルに記録：
- ユーザー登録/削除
- パスワード変更
- ログイン失敗
- 管理者操作

ログ項目：
- 操作日時（UTC）
- 実行ユーザー
- 操作種別
- 影響を受けたユーザー
- IPアドレス

### セキュリティ対策

1. **データ保護**
   - パスワードハッシュにPBKDF2アルゴリズム適用
   - 感情報をログ出力禁止

2. **CSRF対策**
   - クッキーにSameSite=Laxを設定
   - 状態変更操作にAnti-Forgeryトークン要求

3. **ヘッダーセキュリティ**
   - HSTS有効化
   - X-Content-Type-Options設定

### パフォーマンス最適化

1. **JSONファイル操作**
   - メモリキャッシュで読み込み回数を削減
   - 非同期I/Oを徹底適用

2. **ロック戦略**
   - ユーザー単位の細粒度ロックを採用
   - ロック待機タイムアウトを設定（500ms）

3. **バッチ処理**
   - ユーザー一括更新をトランザクション管理
   - 定期的なデータ整合性チェックを実施

### 移行戦略

1. **バージョン管理**
   - JSONスキーマ変更時にバージョニング

2. **フェイルセーフ**
   - 更新前にバックアップ自動作成
   - ユーザーが任意にロールバック

3. **監視項目**
   - ファイルサイズ増加率
   - 認証処理遅延時間
   - ロック競合発生率

### 認証方式の詳細

1. **クッキーベース認証**
   - 認証成功時にHttpOnlyクッキーを発行
   - クッキー名: `Auth.Session`
   - 有効期限: 480分（8時間）
   - セキュリティ設定:
     - HttpOnly: 有効
     - Secure: 有効
     - SameSite: Lax
     - Path: /

2. **セッション管理**
   - サーバーサイドでセッション情報を保持
   - セッションIDは暗号化してクッキーに保存
   - セッション情報:
     - ユーザーID
     - ログイン日時
     - 最終アクセス日時
     - IPアドレス

### バリデーションルール

1. **ユーザー名（UserName）**
   - 最小長: 4文字
   - 最大長: 32文字
   - 使用可能文字: 英数字、アンダースコア、ハイフン
   - 先頭文字: 英字のみ

2. **パスワード**
   - 最小長: 4文字
   - 最大長: 128文字
   - 使用可能文字: すべての文字

3. **表示名（LastName, FirstName）**
   - 最大長: 50文字
   - 使用可能文字: すべての文字

### 多言語対応

1. **リソースファイル構成**
   - `Resources/`
     - `SharedResource.ja.json`
     - `SharedResource.en.json`
     - `ValidationResource.ja.json`
     - `ValidationResource.en.json`

2. **対応言語**
   - 日本語（ja）
   - 英語（en）

3. **実装方法**
   - `IStringLocalizer<T>` を使用
   - デフォルト言語: 日本語
   - Accept-Languageヘッダーで言語切り替え

4. **翻訳対象**
   - エラーメッセージ
   - バリデーションメッセージ
   - システムメッセージ

### テスト戦略

1. **単体テスト**
   - テストフレームワーク: xUnit
   - モックライブラリ: Moq
   - テスト対象:
     - コントローラー
     - サービス
     - バリデーション
     - ユーティリティ

2. **統合テスト**
   - テストフレームワーク: xUnit
   - テスト対象:
     - 認証フロー
     - ユーザー管理
     - エラーハンドリング
   - テストデータ:
     - インメモリデータベース
     - テスト用JSONファイル

3. **テストカバレッジ目標**
   - コードカバレッジ: 80%以上
   - ブランチカバレッジ: 70%以上

4. **テストデータ管理**
   - テスト用JSONファイルをバージョン管理
   - テストデータのリセット機能
   - テスト用の設定ファイル分離

5. **CI/CD統合**
   - プルリクエスト時に自動テスト実行
   - カバレッジレポートの自動生成
   - テスト結果の可視化

### Identityインターフェース実装

1. **IUserPasswordStore<TUser>**
   - パスワードハッシュの生成と検証
   - パスワード変更履歴の管理
   - パスワードリセット機能
   - 実装クラス: `UserPasswordStore`

2. **IUserRoleStore<TUser>**
   - ロール定義:
     - `Admin`: 管理者権限
     - `User`: 一般ユーザー
   - ロール割り当てと削除
   - ロールベースのアクセス制御
   - 実装クラス: `UserRoleStore`

3. **IUserClaimStore<TUser>**
   - クレーム定義:
     - `Permission`: 操作権限
     - `Department`: 所属部署
     - `Custom`: カスタムクレーム
   - クレームの追加と削除
   - クレームベースの認可
   - 実装クラス: `UserClaimStore`

4. **IUserLockoutStore<TUser>**
   - ロックアウト条件:
     - 連続失敗回数: 5回
     - ロックアウト期間: 10分
     - 永久ロックアウト: 5回のロックアウト後
   - ロックアウト状態の管理
   - ロックアウト解除機能
   - 実装クラス: `UserLockoutStore`

#### ストア実装の共通要件

1. **データ永続化**
   - JSONファイルへの保存
   - 非同期I/Oの使用
   - トランザクション管理

2. **キャッシュ戦略**
   - メモリキャッシュの利用
   - キャッシュ無効化条件の定義
   - 分散キャッシュの考慮

3. **エラーハンドリング**
   - ストア固有の例外定義
   - リトライポリシー
   - エラーログ記録

4. **パフォーマンス最適化**
   - バッチ処理の実装
   - 非同期メソッドの提供
   - クエリ最適化
