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
using AuthGate.Shared.Exceptions;
using AuthGate.Shared.Models.Enums;

namespace AuthGate.Shared.Models.Entities;

public class UserAccount
{
    /// <summary>
    /// ユーザーの一意なID
    /// </summary>
    public Guid Id { get; set; }

    /// <summary>
    /// ログインID
    /// </summary>
    public string LoginId { get; set; } = string.Empty;

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
    /// ロール
    /// </summary>
    public Role Role { get; set; } = Role.General;

    /// <summary>
    /// リフレッシュトークン
    /// </summary>
    public string? RefreshToken { get; set; }

    /// <summary>
    /// リフレッシュトークンの有効期限 (UTC)
    /// </summary>
    public DateTime? RefreshTokenExpiryTime { get; set; }

    /// <summary>
    /// 現在の連続ログイン失敗回数。
    /// ログイン成功またはロックアウト時にリセットされる。
    /// </summary>
    public int FailedLoginAttempts { get; set; }

    /// <summary>
    /// アカウントロックアウトの解除日時 (UTC)。
    /// nullの場合、または過去の日時であればロックアウトされていない。
    /// </summary>
    public DateTime? LockoutEnd { get; set; }

    /// <summary>
    /// これまでにアカウントがロックアウトされた総回数。
    /// </summary>
    public int LockoutCount { get; set; }

    /// <summary>
    /// アカウントロックアウトフラグ
    /// </summary>
    public bool IsLocked => LockoutEnd.HasValue && LockoutEnd > DateTime.UtcNow;

    /// <summary>
    /// アカウントバンフラグ
    /// </summary>
    public bool IsBanned { get; set; }

    /// <summary>
    /// パスワードを設定
    /// </summary>
    /// <param name="password">パスワード</param>
    public void SetPassword(string password)
    {
        // パスワードをハッシュ化して保存
        PasswordHash = BCrypt.Net.BCrypt.HashPassword(password);
    }

    /// <summary>
    /// ログイン成功時の処理
    /// </summary>
    public void RecordLoginSuccess(DateTime refreshTokenExpiryTime)
    {
        // ログイン成功時にリセット
        ResetLockoutStatud();
        SetRefreshToken(refreshTokenExpiryTime);
    }

    /// <summary>
    /// リフレッシュトークンを設定
    /// </summary>
    /// <param name="refreshTokenExpiryTime">リフレッシュトークン有効期限</param>
    public void SetRefreshToken(DateTime refreshTokenExpiryTime)
    {
        // リフレッシュトークンを新規作成
        RefreshTokenExpiryTime = refreshTokenExpiryTime;
        RefreshToken = Guid.NewGuid().ToString();
    }

    /// <summary>
    /// リフレッシュトークンを変更
    /// </summary>
    public void ChangeRefreshToken()
    {
        // リフレッシュトークンを変更
        RefreshToken = Guid.NewGuid().ToString();
    }

    /// <summary>
    /// リフレッシュトークンをリセット
    /// </summary>
    public void ResetRefreshToken()
    {
        RefreshToken = null;
        RefreshTokenExpiryTime = null;
    }

    /// <summary>
    /// ログイン失敗時の処理
    /// </summary>
    /// <param name="maxFailedAccessAttempts">最大ログイン失敗回数</param>
    /// <param name="lockoutMinutes">ロックアウト時間</param>
    /// <param name="maxLockoutCount">最大ロックアウト回数</param>
    /// <exception cref="BannedException">バンされているときに発生する例外</exception>
    /// <exception cref="LockoutException">ロックアウトされているときに発生する例外</exception>
    public void RecordLoginFailure(int maxFailedAccessAttempts, int lockoutMinutes, int maxLockoutCount)
    {
        if (IsBanned) { throw new BannedException(); }
        if (IsLocked) { throw new LockoutException((DateTime)LockoutEnd!); }

        FailedLoginAttempts++;

        bool isLockedNow = false;
        if (FailedLoginAttempts >= maxFailedAccessAttempts)
        {
            LockoutEnd = DateTime.UtcNow.AddMinutes(lockoutMinutes);
            LockoutCount++;
            FailedLoginAttempts = 0;
            isLockedNow = true;
        }

        // BANの条件をチェック (ロックアウトのチェック後)
        if (LockoutCount >= maxLockoutCount)
        {
            IsBanned = true;
            throw new BannedException();
        }

        // BANチェック後にロックアウト状態をチェックして例外をスロー
        if (isLockedNow)
        {
            throw new LockoutException((DateTime)LockoutEnd!);
        }
    }

    /// <summary>
    /// アカウントロック解除
    /// </summary>
    public void ResetLockoutStatud()
    {
        FailedLoginAttempts = 0;
        LockoutEnd = null;
        LockoutCount = 0;
    }

    /// <summary>
    /// アカウントバン解除
    /// </summary>
    public void ResetBanStatus()
    {
        IsBanned = false;
        ResetLockoutStatud();
    }
}

```


```CSharp
/* DomainExceptions.cs */
using System;

namespace YourApp.Domain.Exceptions
{
    public abstract class DomainException : Exception
    {
        protected DomainException(string message) : base(message) { }
    }

    public class UserValidationException : DomainException
    {
        public UserValidationException(string message) : base(message) { }
    }

    public class AccountLockoutException : DomainException
    {
        public AccountLockoutException(string message) : base(message) { }
    }
}

/* ValidationAttributes.cs */
using System;
using System.ComponentModel.DataAnnotations;
using System.Text.RegularExpressions;

namespace YourApp.Domain.Validation
{
    public class AlphanumericAttribute : ValidationAttribute
    {
        private static readonly Regex Pattern = new("^[a-zA-Z0-9]+$");
        public override bool IsValid(object? value)
        {
            if (value is string s && Pattern.IsMatch(s)) return true;
            return false;
        }
    }

    public class KanaOnlyAttribute : ValidationAttribute
    {
        private static readonly Regex Pattern = new("^[\u3040-\u309F\u30A0-\u30FFー]+$");
        public override bool IsValid(object? value)
        {
            if (value is string s && Pattern.IsMatch(s)) return true;
            return false;
        }
    }

    public class RomanOnlyAttribute : ValidationAttribute
    {
        private static readonly Regex Pattern = new("^[A-Za-z]+$");
        public override bool IsValid(object? value)
        {
            if (value is string s && Pattern.IsMatch(s)) return true;
            return false;
        }
    }
}

/* User.cs */
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using BCrypt.Net;
using YourApp.Domain.Exceptions;
using YourApp.Domain.Validation;

namespace YourApp.Domain.Models
{
    public class User
    {
        private readonly AuthSettings authSettings;

        [Required]
        public Guid Id { get; private set; }

        [Required(ErrorMessage = "ログインIDは必須入力項目です。")]
        [StringLength(32, MinimumLength = 4, ErrorMessage = "ログインIDは4～32文字で指定してください。")]
        [Alphanumeric(ErrorMessage = "ログインIDは英数字のみ使用できます。")]
        public string LoginId { get; private set; } = string.Empty;

        [Required(ErrorMessage = "ユーザー名は必須入力項目です。")]
        [StringLength(32, MinimumLength = 1, ErrorMessage = "ユーザー名は1～32文字で指定してください。")]
        public string Username { get; private set; } = string.Empty;

        [StringLength(64, MinimumLength = 1, ErrorMessage = "ユーザー名（かな）は1～64文字で指定してください。")]
        [KanaOnly(ErrorMessage = "ユーザー名（かな）はかな文字のみ使用できます。")]
        public string UsernameKana { get; private set; } = string.Empty;

        [StringLength(64, MinimumLength = 1, ErrorMessage = "ユーザー名（ローマ字）は1～64文字で指定してください。")]
        [RomanOnly(ErrorMessage = "ユーザー名（ローマ字）はローマ字のみ使用できます。")]
        public string UsernameRoman { get; private set; } = string.Empty;

        [Required(ErrorMessage = "ロールは必須入力項目です。")]
        public Role Role { get; private set; } = Role.General;

        [Required(ErrorMessage = "パスワードハッシュは必須入力項目です。")]
        public string PasswordHash { get; private set; } = string.Empty;

        public string? RefreshToken { get; private set; }
        public DateTime? RefreshTokenExpiryTime { get; private set; }
        public bool IsLocked => LockoutEnd.HasValue && LockoutEnd.Value > DateTimeOffset.UtcNow;
        public bool IsBanned { get; private set; }
        public int FailedLoginAttempts { get; private set; }
        public DateTime? LockoutEnd { get; private set; }
        public int LockoutCount { get; private set; }

        public User(
            string loginId,
            string username,
            string password,
            Role role,
            AuthSettings authSettings)
        {
            this.authSettings = authSettings ?? throw new ArgumentNullException(nameof(authSettings));

            if (string.IsNullOrWhiteSpace(loginId))
                throw new ArgumentException("ログインIDが無効な値です。", nameof(loginId));
            if (string.IsNullOrWhiteSpace(username))
                throw new ArgumentException("ユーザー名が無効な値です。", nameof(username));
            if (string.IsNullOrWhiteSpace(password))
                throw new ArgumentException("パスワードが空です。", nameof(password));

            Id = Guid.NewGuid();
            LoginId = loginId;
            Username = username;
            PasswordHash = BCrypt.HashPassword(password);
            Role = role;

            ValidateUser(this);
        }

        public void SetUsername(string newUsername)
        {
            Username = newUsername;
            ValidateUser(this);
        }

        public void SetUsernameKana(string newUsernameKana)
        {
            UsernameKana = newUsernameKana;
            ValidateUser(this);
        }

        public void SetUsernameRoman(string newUsernameRoman)
        {
            UsernameRoman = newUsernameRoman;
            ValidateUser(this);
        }

        public void SetRole(Role newRole, bool hasPermission = false)
        {
            if (!hasPermission)
                throw new InvalidOperationException("ロールの変更権限がありません。");

            Role = newRole;
            ValidateUser(this);
        }

        public void SetPassword(string newPassword)
        {
            if (string.IsNullOrWhiteSpace(newPassword))
                throw new ArgumentException("パスワードは必須入力項目です。", nameof(newPassword));

            PasswordHash = BCrypt.HashPassword(newPassword);
            ValidateUser(this);

            ResetLockoutStatus();
            ClearRefreshToken();
        }

        public void SetRefreshToken(string token, DateTime expiryTime)
        {
            if (string.IsNullOrWhiteSpace(token))
                throw new ArgumentException("トークンは必須入力項目です。", nameof(token));
            if (expiryTime <= DateTime.UtcNow)
                throw new ArgumentOutOfRangeException(nameof(expiryTime), "有効期限は現在の日時よりも未来である必要があります。");

            RefreshToken = token;
            RefreshTokenExpiryTime = expiryTime.Kind == DateTimeKind.Unspecified
                ? DateTime.SpecifyKind(expiryTime, DateTimeKind.Utc)
                : expiryTime.ToUniversalTime();
        }

        public void ClearRefreshToken()
        {
            RefreshToken = null;
            RefreshTokenExpiryTime = null;
        }

        private void ValidateUser(User user)
        {
            var context = new ValidationContext(user);
            var results = new List<ValidationResult>();
            bool isValid = Validator.TryValidateObject(user, context, results, true);

            if (!isValid)
            {
                foreach (var result in results)
                {
                    throw new UserValidationException(result.ErrorMessage!);
                }
            }
        }

        public bool RecordLoginFailure()
        {
            if (IsBanned || IsLocked)
                return false;

            FailedLoginAttempts++;
            bool locked = false;

            if (FailedLoginAttempts >= authSettings.MaxFailedAccessAttempts)
            {
                LockoutEnd = DateTime.UtcNow.AddMinutes(authSettings.LockoutMinutes);
                LockoutCount++;
                FailedLoginAttempts = 0;
                locked = true;
            }

            if (LockoutCount >= authSettings.MaxLockoutCount)
            {
                IsBanned = true;
            }

            return locked;
        }

        public void ResetLockoutStatus()
        {
            FailedLoginAttempts = 0;
            LockoutEnd = null;
            LockoutCount = 0;
            IsBanned = false;
        }
    }
}

```
