## 概要
本仕様書は、ASP.NET Core Web API を使用して構築される JWT (JSON Web Token) 認証機能を提供するAPIについて定義します。このAPIは、ユーザーの認証を行い、有効なJWTを発行することで、保護されたAPIエンドポイントへの安全なアクセスを可能にします。

---
## 技術スタック
本APIは以下の技術スタックを使用して開発されます。特に、ASP.NET Coreの認証・認可メカニズムを最大限に活用します。
- フレームワーク: ASP.NET Core Web API
  - モダンで高性能なクロスプラットフォームフレームワーク。Web APIの構築に最適です。RESTfulな設計を容易に実現できます。
- プログラミング言語: C#
  - ASP.NET Coreの主要言語。強力な型付けと豊富なライブラリを備えています。
- 認証・認可ミドルウェア:
  - `Microsoft.AspNetCore.Authentication.JwtBearer`: JWT認証の核心となるライブラリです。受信したHTTPリクエストの`Authorization`ヘッダーからJWTを抽出し、検証を行います。検証が成功した場合、`HttpContext.User`にユーザーのPrincipal（認証されたユーザー情報やClaims）を設定します。
  - `Microsoft.AspNetCore.Authentication`: 認証スキーム（JWT Bearerなど）を管理するための基本ライブラリです。
  - `Microsoft.AspNetCore.Authorization`: リソース（APIエンドポイント）へのアクセス制御（認可）を行います。`[Authorize]`属性を使用して、特定のエンドポイントへのアクセスに認証が必要であることを指定します。
- Identity管理:
  - `Microsoft.AspNetCore.Identity`: ユーザー、パスワード、ロール、Claimsなどの管理を抽象化するフレームワークです。パスワードのハッシュ化やユーザーストアとの連携を容易に行えます。カスタムユーザープロパティやストアを拡張することも可能です。JWT認証と連携させることで、ユーザー認証部分の実装負荷を軽減できます。
  - `Microsoft.AspNetCore.Identity.EntityFrameworkCore`: Entity Framework Core を使用してIdentity情報をリレーショナルデータベースに保存するためのプロバイダーです。
- データベース:
  - Entity Framework Core と連携可能なリレーショナルデータベース（例: SQL Server, PostgreSQL, MySQL, SQLiteなど）。ユーザー情報やRefresh Token（後述）を格納します。
- ORM (Object-Relational Mapping):
  - Entity Framework Core: .NET向けの推奨ORMです。データベース操作をC#オブジェクトとして扱うことができます。Identityのユーザー情報の永続化に使用します。
- JWTライブラリ:
  - `System.IdentityModel.Tokens.Jwt`: JWTの生成、署名、検証を行う低レベルライブラリです。`Microsoft.AspNetCore.Authentication.JwtBearer`はこのライブラリを利用してJWTの処理を行います。
- 設定管理: `appsettings.json` および環境変数
  - JWTの署名に使用するシークレットキー、発行者（Issuer）、受け取り手（Audience）、トークンの有効期限などの設定情報を管理します。ASP.NET Coreの強力な設定プロバイダー機能を利用します。
- DI (Dependency Injection): ASP.NET Core組み込みのDIコンテナ
  - アプリケーション内のコンポーネント間の依存関係を管理します。`UserManager<TUser>`, `SignInManager<TUser>`, データベースコンテキストなどがDIによってControllerやServiceに提供されます。
- ロギング: ASP.NET Core組み込みのロギングフレームワーク
  - 認証処理やエラー発生時のログを出力し、デバッグや運用監視に役立てます。`ILogger`インターフェースを使用します。
- APIドキュメンテーション: Swashbuckle (OpenAPI/Swagger) (推奨)
  - APIエンドポイント、リクエスト/レスポンスモデル、認証方法などを記述したドキュメントを自動生成します。Swagger UIを通じてAPIのテストも容易に行えます。
---
## APIエンドポイント
以下のエンドポイントを定義します。

### Auth
- **POST /api/auth/login**
    - **機能:** ログイン認証
    - **Request DTO:** `LoginRequestDto` (例: ログインID、パスワードを含む)
    - **Response DTO:** `LoginResponseDto` (例: アクセストークン、リフレッシュトークン、ユーザー情報の一部を含む)
- **POST /api/auth/refresh-token**
    - **機能:** リフレッシュトークンを利用したアクセストークンのリフレッシュ
    - **Request DTO:** `RefreshTokenRequestDto` (例: リフレッシュトークンを含む)
    - **Response DTO:** `TokenResponseDto` (例: 新しいアクセストークンを含む)
- **GET /api/auth/comfirm**
    - **機能:** 現在認証中のユーザー情報を取得
    - **Request DTO:** なし
    - **Response DTO:** なし (成功を示すHTTPステータスコード 204 No Content)

### User
- **POST /api/user/register**
    - **機能:** ユーザーの登録(管理者)
    - **Request DTO:** `UserRegisterRequestDto` (例: ログインID、パスワード、ユーザー名、ロールなどを含む)
    - **Response DTO:** `UserResponseDto` (登録されたユーザー情報)
- **GET /api/user/users**
    - **機能:** 全ユーザーの取得(管理者)
    - **Request DTO:** なし (ページネーションやフィルタリングのためのクエリパラメータはあり得る)
    - **Response DTO:** `UserListResponseDto` (ユーザー情報のリスト) または `List<UserResponseDto>`
- **GET /api/user/{userId:Guid}**
    - **機能:** ユーザーIDを指定してユーザーを取得(管理者または自身)
    - **Request DTO:** なし
    - **Response DTO:** `UserResponseDto` (指定されたユーザー情報)
- **GET /api/user/login-id/{loginId}**
    - **機能:** ログインIDを指定してユーザーの取得(管理者または自身)
    - **Request DTO:** なし
    - **Response DTO:** `UserResponseDto` (指定されたユーザー情報)
- **POST /api/user/search**
    - **機能:** ユーザー名等を利用してユーザーを取得(管理者または自身)
    - **Request DTO:** `UserSearchRequestDto` (例: 検索キーワード、検索対象フィールドなどを含む)
    - **Response DTO:** `UserListResponseDto` (検索結果のユーザー情報リスト) または `List<UserResponseDto>`
- **PATCH /api/user/{userId:Guid}**
    - **機能:** パスワードおよびロール以外のユーザー情報の変更(管理者または自身)
    - **Request DTO:** `UserUpdateRequestDto` (例: ユーザー名、メールアドレスなど変更可能な情報を含む)
    - **Response DTO:** `UserResponseDto` (更新後のユーザー情報)
- **PATCH /api/user/{userId:Guid}/password**
    - **機能:** パスワードの変更(管理者または自身)
    - **Request DTO:** `UserPasswordUpdateRequestDto` (例: 現在のパスワード（本人変更時）、新しいパスワードを含む)
    - **Response DTO:** なし (成功を示すHTTPステータスコード 200 OK or 204 No Content)
- **PATCH /api/user/{userId:Guid}/role**
    - **機能:** ロールの変更(管理者)
    - **Request DTO:** `UserRoleUpdateRequestDto` (例: 新しいロール情報を含む)
    - **Response DTO:** `UserResponseDto` (更新後のユーザー情報、または成功を示すステータスコード)
- **DELETE /api/user/{userId:Guid}**
    - **機能:** ユーザーの削除(管理者)
    - **Request DTO:** なし
    - **Response DTO:** なし (成功を示すHTTPステータスコード 204 No Content)

## Data Transfer Object

以下のデータ転送オブジェクトを定義します。これらのDTOは、APIリクエストのペイロードやレスポンスボディとして使用され、クライアントとサーバー間で構造化されたデータを効率的にやり取りすることを目的としています。

- **`LoginRequestDto`**
    - **目的・役割:** ユーザーがログイン認証を行う際に、クライアントからサーバーへ送信するログイン情報（例: ログインID、パスワード）を格納します。
    - **主な利用エンドポイント:** `POST /api/auth/login`
- **`LoginResponseDto`**
    - **目的・役割:** ログイン認証成功時に、サーバーからクライアントへ返却される情報（例: アクセストークン、リフレッシュトークン、ユーザー情報の一部）を格納します。
    - **主な利用エンドポイント:** `POST /api/auth/login`
- **`RefreshTokenRequestDto`**
    - **目的・役割:** アクセストークンをリフレッシュする際に、クライアントからサーバーへ送信するリフレッシュトークン情報を格納します。
    - **主な利用エンドポイント:** `POST /api/auth/refresh-token`
- **`TokenResponseDto`**
    - **目的・役割:** アクセストークンのリフレッシュ成功時に、サーバーからクライアントへ返却される新しいアクセストークン情報を格納します。
    - **主な利用エンドポイント:** `POST /api/auth/refresh-token`
- **`UserRegisterRequestDto`**
    - **目的・役割:** 管理者が新しいユーザーを登録する際に、クライアントからサーバーへ送信するユーザー情報（例: ログインID、パスワード、ユーザー名、ロールなど）を格納します。
    - **主な利用エンドポイント:** `POST /api/user/register`
- **`UserResponseDto`**
    - **目的・役割:** サーバーからクライアントへ単一のユーザー情報を返却する際に使用します。ユーザー登録時、ユーザー情報取得時、ユーザー情報更新時などに利用され、通常、パスワードのような機密情報は含みません。
    - **主な利用エンドポイント:** `POST /api/user/register`, `GET /api/user/{userId:Guid}`, `GET /api/user/login-id/{loginId}`, `PATCH /api/user/{userId:Guid}`, `PATCH /api/user/{userId:Guid}/role`
- **`UserListResponseDto`**
    - **目的・役割:** サーバーからクライアントへ複数のユーザー情報のリストを返却する際に使用します。全ユーザー取得時やユーザー検索結果などで利用され、ページネーション情報を含むこともあります。実質的には `List<UserResponseDto>` とページネーション情報を組み合わせた形になることが多いです。
    - **主な利用エンドポイント:** `GET /api/user/users`, `POST /api/user/search`
- **`UserSearchRequestDto`**
    - **目的・役割:** ユーザーを検索する際に、クライアントからサーバーへ送信する検索条件（例: ユーザー名の一部、メールアドレスなど）を格納します。
    - **主な利用エンドポイント:** `POST /api/user/search`
- **`UserUpdateRequestDto`**
    - **目的・役割:** 既存ユーザーの情報（パスワードおよびロール以外。例: ユーザー名、メールアドレスなど）を更新する際に、クライアントからサーバーへ送信する更新情報を格納します。
    - **主な利用エンドポイント:** `PATCH /api/user/{userId:Guid}`
- **`UserPasswordUpdateRequestDto`**
    - **目的・役割:** ユーザーのパスワードを変更する際に、クライアントからサーバーへ送信する情報（例: 現在のパスワード（本人変更時）、新しいパスワード）を格納します。
    - **主な利用エンドポイント:** `PATCH /api/user/{userId:Guid}/password`
- **`UserRoleUpdateRequestDto`**
    - **目的・役割:** 管理者がユーザーのロールを変更する際に、クライアントからサーバーへ送信する新しいロール情報を格納します。
    - **主な利用エンドポイント:** `PATCH /api/user/{userId:Guid}/role`

## Entities

### User
```CSharp
// User.cs
using System;

public class User
{
    /// <summary>
    /// ユーザーの一意なID
    /// </summary>
    public Guid Id { get; private set; }

    /// <summary>
    /// ログインID
    /// </summary>
    public string LoginId { get; private set; } = string.Empty;

    /// <summary>
    /// ユーザー名
    /// </summary>
    public string Username { get; set; } = string.Empty;

    /// <summary>
    /// ユーザー名（かな）
    /// </summary>
	public string UsernameKana { get; set; } = string.Empty;

    /// <summary>
    /// ユーザー名（ローマ字）
    /// </summary>
	public string UsernameRoman { get; set; } = string.Empty;

    /// <summary>
    /// パスワードハッシュ
    /// </summary>
    public string PasswordHash { get; private set; } = string.Empty;
	
    /// <summary>
    /// リフレッシュトークン
    /// </summary>
    public Guid? RefreshToken { get; private set; }

    /// <summary>
    /// リフレッシュトークンの有効期限 (UTC)
    /// </summary>
    public DateTime? RefreshTokenExpiryTime { get; private set; }

    /// <summary>
    /// 現在の連続ログイン失敗回数。
    /// ログイン成功またはロックアウト時にリセットされる。
    /// </summary>
    public int FailedLoginAttempts { get; private set; }

    /// <summary>
    /// アカウントロックアウトの解除日時 (UTC)。
    /// nullの場合、または過去の日時であればロックアウトされていない。
    /// </summary>
    public DateTimeOffset? LockoutEnd { get; private set; }

    /// <summary>
    /// これまでにアカウントがロックアウトされた総回数。
    /// </summary>
    public int LockoutCount { get; private set; }

    /// <summary>
    /// ユーザーアカウントがアクティブかどうか。
    /// （例: 論理削除、管理者による一時停止など）
    /// </summary>
    public bool IsActive { get; set; }


    /// <summary>
    /// アカウントが現在一時的にロックアウトされているかどうかを確認します。
    /// </summary>
    public bool IsTemporarilyLockedOut => LockoutEnd.HasValue && LockoutEnd.Value > DateTimeOffset.UtcNow;

    /// <summary>
    /// コンストラクタ。
    /// </summary>
    /// <param name="loginId">ログインID。</param>
    /// <param name="username">ユーザー名。</param>
    /// <param name="hashedPassword">ハッシュ化されたパスワード。</param>
    /// <param name="passwordSalt">パスワードソルト。</param>
    public User(string loginId, string username, string hashedPassword, string passwordSalt)
    {
        if (string.IsNullOrWhiteSpace(loginId))
            throw new ArgumentException("LoginId cannot be empty.", nameof(loginId));
        if (string.IsNullOrWhiteSpace(username))
            throw new ArgumentException("Username cannot be empty.", nameof(username));
        if (string.IsNullOrWhiteSpace(hashedPassword))
            throw new ArgumentException("HashedPassword cannot be empty.", nameof(hashedPassword));
        if (string.IsNullOrWhiteSpace(passwordSalt))
            throw new ArgumentException("PasswordSalt cannot be empty.", nameof(passwordSalt));

        Id = Guid.NewGuid();
        LoginId = loginId;
        Username = username;
        HashedPassword = hashedPassword;
        PasswordSalt = passwordSalt;

        FailedLoginAttempts = 0;
        LockoutCount = 0;
        IsActive = true; // デフォルトはアクティブ
    }

    /// <summary>
    /// パスワードを設定または更新します。
    /// </summary>
    /// <param name="newHashedPassword">新しいハッシュ化されたパスワード。</param>
    /// <param name="newPasswordSalt">新しいパスワードソルト。</param>
    public void SetPassword(string newHashedPassword, string newPasswordSalt)
    {
        if (string.IsNullOrWhiteSpace(newHashedPassword))
            throw new ArgumentException("NewHashedPassword cannot be empty.", nameof(newHashedPassword));
        if (string.IsNullOrWhiteSpace(newPasswordSalt))
            throw new ArgumentException("NewPasswordSalt cannot be empty.", nameof(newPasswordSalt));

        HashedPassword = newHashedPassword;
        PasswordSalt = newPasswordSalt;
        // パスワード変更時は、セキュリティのため関連する状態をリセットすることが推奨されます。
        ResetLockoutStatus(); // 失敗回数やロックアウト状態をクリア
        ClearRefreshToken();  // 既存のセッションも無効化することが望ましい場合がある
    }

    /// <summary>
    /// リフレッシュトークンとその有効期限を設定します。
    /// </summary>
    /// <param name="token">リフレッシュトークン。</param>
    /// <param name="expiryTime">有効期限 (UTC)。</param>
    public void SetRefreshToken(string token, DateTime expiryTime)
    {
        if (string.IsNullOrWhiteSpace(token))
            throw new ArgumentException("Token cannot be empty.", nameof(token));
        if (expiryTime <= DateTime.UtcNow)
            throw new ArgumentOutOfRangeException(nameof(expiryTime), "Expiry time must be in the future.");

        RefreshToken = token;
        RefreshTokenExpiryTime = expiryTime.Kind == DateTimeKind.Unspecified
            ? DateTime.SpecifyKind(expiryTime, DateTimeKind.Utc) // 未指定ならUTCとして扱う
            : expiryTime.ToUniversalTime();
    }

    /// <summary>
    /// リフレッシュトークンをクリアします。
    /// </summary>
    public void ClearRefreshToken()
    {
        RefreshToken = null;
        RefreshTokenExpiryTime = null;
    }

    /// <summary>
    /// ログイン成功を記録します。
    /// 連続失敗回数をリセットします。
    /// </summary>
    public void RecordLoginSuccess()
    {
        FailedLoginAttempts = 0;
        // 既存のロックアウトが自然解除されるのを待つか、
        // ログイン成功で一時ロックアウトを解除するかはポリシーによります。
        // ここでは失敗回数のみリセットします。
        // LockoutEnd = null; // もし即時解除する場合
    }

    /// <summary>
    /// ログイン失敗を記録し、必要に応じてアカウントをロックアウトします。
    /// </summary>
    /// <param name="settings">認証設定。</param>
    public void RecordLoginFailure(AuthSettings settings)
    {
        if (settings == null) throw new ArgumentNullException(nameof(settings));

        // 永久ロックされているか、または既に一時ロックアウト中の場合は、カウンタを更新しない
        if (IsPermanentlyLocked(settings) || IsTemporarilyLockedOut)
        {
            return;
        }

        FailedLoginAttempts++;

        if (FailedLoginAttempts >= settings.MaxFailedAccessAttemptsBeforeLockout)
        {
            LockoutEnd = DateTimeOffset.UtcNow.AddMinutes(settings.DefaultLockoutMinutes);
            LockoutCount++;
            FailedLoginAttempts = 0; // ロックアウト後は失敗カウントをリセット
        }
    }

    /// <summary>
    /// アカウントが永久にロックアウトされているかどうかを確認します。
    /// </summary>
    /// <param name="settings">認証設定。</param>
    /// <returns>永久ロックされていれば true、そうでなければ false。</returns>
    public bool IsPermanentlyLocked(AuthSettings settings)
    {
        if (settings == null) throw new ArgumentNullException(nameof(settings));
        // MaxLockoutCountForPermanentLockout が0以下なら永久ロック機能は無効と解釈
        if (settings.MaxLockoutCountForPermanentLockout <= 0)
        {
            return false;
        }
        return LockoutCount >= settings.MaxLockoutCountForPermanentLockout;
    }

    /// <summary>
    /// アカウントのロックアウト状態（失敗回数、ロックアウト終了日時、ロックアウト回数）をリセットします。
    /// 主にパスワードリセット成功時や管理者によるロック解除時に使用します。
    /// </summary>
    public void ResetLockoutStatus()
    {
        FailedLoginAttempts = 0;
        LockoutEnd = null;
        // LockoutCountをリセットするかどうかはポリシーによります。
        // 永久ロックのカウントもリセットする場合は LockoutCount = 0;
    }
}
```