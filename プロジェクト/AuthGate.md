# ASP.NET Core Web API 認証システム設計計画書

## プロジェクト概要

LoginIDを使用したCookieベース認証を実装するASP.NET Core Web APIプロジェクトを設計・構築します。システムはPostgreSQLをデータベースとして使用し、.NET 9フレームワーク上に構築されます。

## 技術スタック

- **フレームワーク**: .NET 9
- **データベース**: PostgreSQL
- **認証方式**: Cookieベース認証
- **ORM**: Entity Framework Core
- **認証基盤**: ASP.NET Core Identity (カスタマイズ)

## システムアーキテクチャ

システムは以下の層構造で設計します：

1. **プレゼンテーション層**
    - Controllers (AuthController, UserController)
    - DTOs (Data Transfer Objects)
    - Filters (認証・認可フィルター)

2. **ビジネスロジック層**
    - Services (認証・ユーザー管理)
    - Managers (ドメインロジック管理)

3. **データアクセス層**
    - Repositories
    - Stores (Identity関連のデータアクセス)
    - Entity Framework Core DbContext
    - Entities

4. **インフラストラクチャ層**
    - データベース設定
    - ロギング
    - 例外処理

## データモデル

### 1. UserAccount

```csharp
using Microsoft.AspNetCore.Identity;
using System.ComponentModel;
namespace AuthGate.Models.Entities;

/// <summary>
/// ユーザーアカウントクラス
/// </summary>
public class UserAccount : IdentityUser
{
    /// <summary>
    /// ログインID
    /// </summary>
    [Description("ログインID")]
    public string LoginId { get; set; } = string.Empty;
    
    /// <summary>
    /// ユーザー名(かな)
    /// </summary>
    [Description("ユーザー名(かな)")]
    public string? UserNameKana { get; set; }
    
    /// <summary>
    /// ユーザー名(ローマ字)
    /// </summary>
    [Description("ユーザー名(ローマ字)")]
    public string? UserNameRoman { get; set; }
    
    /// <summary>
    /// ユーザーが所属するロール
    /// </summary>
    [Description("ロール")]
    public virtual ICollection<IdentityRole>? Roles { get; set; }
}
```


## 主要コンポーネント設計

### 1. Store層 (Identity用データアクセス)

Identity関連の操作に特化したデータアクセス層を提供します。

#### UserStore

```csharp
public class UserAccountStore : 
    IUserStore<UserAccount>, 
    IUserPasswordStore<UserAccount>,
    IUserRoleStore<UserAccount>
{
    private readonly AppDbContext _context;
    
    public UserAccountStore(AppDbContext context)
    {
        _context = context;
    }
    
    // IUserStore 実装
    public Task<IdentityResult> CreateAsync(UserAccount user, CancellationToken cancellationToken = default);
    public Task<IdentityResult> UpdateAsync(UserAccount user, CancellationToken cancellationToken = default);
    public Task<IdentityResult> DeleteAsync(UserAccount user, CancellationToken cancellationToken = default);
    public Task<UserAccount> FindByIdAsync(string userId, CancellationToken cancellationToken = default);
    public Task<UserAccount> FindByNameAsync(string normalizedUserName, CancellationToken cancellationToken = default);
    
    // IUserPasswordStore 実装
    public Task SetPasswordHashAsync(UserAccount user, string passwordHash, CancellationToken cancellationToken = default);
    public Task<string> GetPasswordHashAsync(UserAccount user, CancellationToken cancellationToken = default);
    public Task<bool> HasPasswordAsync(UserAccount user, CancellationToken cancellationToken = default);
    
    // IUserRoleStore 実装
    public Task AddToRoleAsync(UserAccount user, string roleName, CancellationToken cancellationToken = default);
    public Task RemoveFromRoleAsync(UserAccount user, string roleName, CancellationToken cancellationToken = default);
    public Task<IList<string>> GetRolesAsync(UserAccount user, CancellationToken cancellationToken = default);
    public Task<bool> IsInRoleAsync(UserAccount user, string roleName, CancellationToken cancellationToken = default);
    
    // カスタムメソッド
    public Task<UserAccount> FindByLoginIdAsync(string loginId, CancellationToken cancellationToken = default);
}
```

#### RoleStore

```csharp
public class RoleStore : IRoleStore<IdentityRole>
{
    private readonly AppDbContext _context;
    
    public RoleStore(AppDbContext context)
    {
        _context = context;
    }
    
    public Task<IdentityResult> CreateAsync(Role role, CancellationToken cancellationToken = default);
    public Task<IdentityResult> UpdateAsync(Role role, CancellationToken cancellationToken = default);
    public Task<IdentityResult> DeleteAsync(Role role, CancellationToken cancellationToken = default);
    public Task<Role> FindByIdAsync(string roleId, CancellationToken cancellationToken = default);
    public Task<Role> FindByNameAsync(string normalizedRoleName, CancellationToken cancellationToken = default);
}
```

### 2. リポジトリ層

Store層の上に構築され、より高レベルなデータアクセス操作を提供します。

#### IUserRepository

```csharp
public interface IUserRepository
{
    Task<UserAccount> GetByLoginIdAsync(string loginId);
    Task<UserAccount> GetByIdAsync(string id);
    Task<List<UserAccount>> GetAllAsync();
    Task<bool> CreateAsync(UserAccount user, string password);
    Task<bool> UpdateAsync(UserAccount user);
    Task<bool> DeleteAsync(string id);
    Task<bool> CheckPasswordAsync(UserAccount user, string password);
    Task<bool> AddToRoleAsync(UserAccount user, string roleName);
    Task<bool> RemoveFromRoleAsync(UserAccount user, string roleName);
    Task<IList<string>> GetRolesAsync(UserAccount user);
    Task<bool> IsInRoleAsync(UserAccount user, string roleName);
}
```

#### IRoleRepository

```csharp
public interface IRoleRepository
{
    Task<Role> GetByIdAsync(string id);
    Task<Role> GetByNameAsync(string name);
    Task<List<Role>> GetAllAsync();
    Task<bool> CreateAsync(Role role);
    Task<bool> UpdateAsync(Role role);
    Task<bool> DeleteAsync(string id);
    Task<List<UserAccount>> GetUsersInRoleAsync(string roleName);
}
```

### 3. Manager層

標準のUserManager/RoleManagerにカスタムロジックを追加します。

#### UserManager<UserAccount>のカスタマイズ

```csharp
public class UserAccountManager : UserManager<UserAccount>
{
    public UserAccountManager(
        IUserStore<UserAccount> store,
        IOptions<IdentityOptions> optionsAccessor,
        IPasswordHasher<UserAccount> passwordHasher,
        IEnumerable<IUserValidator<UserAccount>> userValidators,
        IEnumerable<IPasswordValidator<UserAccount>> passwordValidators,
        ILookupNormalizer keyNormalizer,
        IdentityErrorDescriber errors,
        IServiceProvider services,
        ILogger<UserManager<UserAccount>> logger) 
        : base(store, optionsAccessor, passwordHasher, userValidators, 
              passwordValidators, keyNormalizer, errors, services, logger)
    {
    }
    
    public async Task<UserAccount> FindByLoginIdAsync(string loginId)
    {
        var store = Store as UserAccountStore;
        if (store == null)
        {
            throw new NotSupportedException("StoreがUserAccountStoreを実装していません");
        }
        
        return await store.FindByLoginIdAsync(loginId, CancellationToken.None);
    }
    
    public async Task<IdentityResult> CreateUserWithRolesAsync(UserAccount user, string password, IEnumerable<string> roles)
    {
        var result = await CreateAsync(user, password);
        if (result.Succeeded && roles != null && roles.Any())
        {
            foreach (var role in roles)
            {
                await AddToRoleAsync(user, role);
            }
        }
        return result;
    }
}
```

#### RoleManager<Role>のカスタマイズ

```csharp
public class ApplicationRoleManager : RoleManager<Role>
{
    public ApplicationRoleManager(
        IRoleStore<Role> store,
        IEnumerable<IRoleValidator<Role>> roleValidators,
        ILookupNormalizer keyNormalizer,
        IdentityErrorDescriber errors,
        ILogger<RoleManager<Role>> logger) 
        : base(store, roleValidators, keyNormalizer, errors, logger)
    {
    }
    
    public async Task<List<UserAccount>> GetUsersInRoleAsync(string roleName)
    {
        var role = await FindByNameAsync(roleName);
        if (role == null)
        {
            return new List<UserAccount>();
        }
        
        // ここでユーザー検索の実装...
        // 実際の実装はDbContextを使用して関連付けを検索する必要があります
        return new List<UserAccount>();
    }
}
```

### 4. サービス層

Manager層を使用して、より高レベルの操作を提供します。

#### IAuthService

```csharp
public interface IAuthService
{
    Task<bool> ValidateUserAsync(string loginId, string password);
    Task<bool> SignInAsync(string loginId, bool rememberMe = false);
    Task SignOutAsync();
    Task<UserAccount> GetCurrentUserAsync();
    Task<bool> IsInRoleAsync(UserAccount user, string roleName);
    Task<IList<string>> GetUserRolesAsync(UserAccount user);
}
```

#### AuthService実装

```csharp
public class AuthService : IAuthService
{
    private readonly UserAccountManager _userManager;
    private readonly SignInManager<UserAccount> _signInManager;
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public AuthService(
        UserAccountManager userManager,
        SignInManager<UserAccount> signInManager,
        IHttpContextAccessor httpContextAccessor)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _httpContextAccessor = httpContextAccessor;
    }
    
    public async Task<bool> ValidateUserAsync(string loginId, string password)
    {
        var user = await _userManager.FindByLoginIdAsync(loginId);
        if (user == null)
        {
            return false;
        }
        
        return await _userManager.CheckPasswordAsync(user, password);
    }
    
    public async Task<bool> SignInAsync(string loginId, bool rememberMe = false)
    {
        var user = await _userManager.FindByLoginIdAsync(loginId);
        if (user == null)
        {
            return false;
        }
        
        await _signInManager.SignInAsync(user, rememberMe);
        return true;
    }
    
    public async Task SignOutAsync()
    {
        await _signInManager.SignOutAsync();
    }
    
    public async Task<UserAccount> GetCurrentUserAsync()
    {
        var user = await _userManager.GetUserAsync(_httpContextAccessor.HttpContext.User);
        return user;
    }
    
    public async Task<bool> IsInRoleAsync(UserAccount user, string roleName)
    {
        return await _userManager.IsInRoleAsync(user, roleName);
    }
    
    public async Task<IList<string>> GetUserRolesAsync(UserAccount user)
    {
        return await _userManager.GetRolesAsync(user);
    }
}
```

#### IUserService

```csharp
public interface IUserService
{
    Task<UserAccount> GetByLoginIdAsync(string loginId);
    Task<UserAccount> GetByIdAsync(string id);
    Task<List<UserAccount>> GetAllAsync();
    Task<bool> CreateUserAsync(UserAccount user, string password, IList<string> roles);
    Task<bool> UpdateUserAsync(UserAccount user);
    Task<bool> DeleteUserAsync(string id);
    Task<bool> AddToRoleAsync(string userId, string roleName);
    Task<bool> RemoveFromRoleAsync(string userId, string roleName);
    Task<IList<string>> GetUserRolesAsync(string userId);
}
```

#### IRoleService

```csharp
public interface IRoleService
{
    Task<Role> GetRoleByIdAsync(string id);
    Task<Role> GetRoleByNameAsync(string name);
    Task<List<Role>> GetAllRolesAsync();
    Task<bool> CreateRoleAsync(Role role);
    Task<bool> UpdateRoleAsync(Role role);
    Task<bool> DeleteRoleAsync(string id);
    Task<List<UserAccount>> GetUsersInRoleAsync(string roleName);
}
```

### 5. コントローラ層

#### AuthController

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;
    
    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }
    
    // Login
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginDto model)
    {
        // バリデーションはModelStateで自動的に処理される
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        if (await _authService.ValidateUserAsync(model.LoginId, model.Password))
        {
            await _authService.SignInAsync(model.LoginId, model.RememberMe);
            return Ok(new { message = "ログイン成功" });
        }
        
        return Unauthorized(new { message = "ログインIDまたはパスワードが正しくありません" });
    }

    // Logout
    [HttpPost("logout")]
    [Authorize]
    public async Task<IActionResult> Logout()
    {
        await _authService.SignOutAsync();
        return Ok(new { message = "ログアウト成功" });
    }

    // Get current user info
    [HttpGet("me")]
    [Authorize]
    public async Task<IActionResult> GetCurrentUser()
    {
        var user = await _authService.GetCurrentUserAsync();
        if (user == null)
        {
            return NotFound();
        }
        
        var roles = await _authService.GetUserRolesAsync(user);
        
        return Ok(new UserInfoDto
        {
            Id = user.Id,
            LoginId = user.LoginId,
            UserName = user.UserName,
            UserNameKana = user.UserNameKana,
            UserNameRoman = user.UserNameRoman,
            Roles = roles.ToList()
        });
    }
}
```

#### UserController

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize(Roles = "Admin")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IRoleService _roleService;
    
    public UserController(IUserService userService, IRoleService roleService)
    {
        _userService = userService;
        _roleService = roleService;
    }
    
    // Get all users
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var users = await _userService.GetAllAsync();
        var userDtos = new List<UserInfoDto>();
        
        foreach (var user in users)
        {
            var roles = await _userService.GetUserRolesAsync(user.Id);
            userDtos.Add(new UserInfoDto
            {
                Id = user.Id,
                LoginId = user.LoginId,
                UserName = user.UserName,
                UserNameKana = user.UserNameKana,
                UserNameRoman = user.UserNameRoman,
                Roles = roles.ToList()
            });
        }
        
        return Ok(userDtos);
    }

    // Get user by ID
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(string id)
    {
        var user = await _userService.GetByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }
        
        var roles = await _userService.GetUserRolesAsync(id);
        
        return Ok(new UserInfoDto
        {
            Id = user.Id,
            LoginId = user.LoginId,
            UserName = user.UserName,
            UserNameKana = user.UserNameKana,
            UserNameRoman = user.UserNameRoman,
            Roles = roles.ToList()
        });
    }

    // Create user
    [HttpPost]
    public async Task<IActionResult> Create([FromBody] CreateUserDto model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        var user = new UserAccount
        {
            LoginId = model.LoginId,
            UserName = model.UserName,
            UserNameKana = model.UserNameKana,
            UserNameRoman = model.UserNameRoman
        };
        
        var result = await _userService.CreateUserAsync(user, model.Password, model.Roles);
        if (result)
        {
            return CreatedAtAction(nameof(GetById), new { id = user.Id }, null);
        }
        
        return BadRequest(new { message = "ユーザー作成に失敗しました" });
    }

    // Update user
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(string id, [FromBody] UpdateUserDto model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        var user = await _userService.GetByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }
        
        user.UserName = model.UserName ?? user.UserName;
        user.UserNameKana = model.UserNameKana ?? user.UserNameKana;
        user.UserNameRoman = model.UserNameRoman ?? user.UserNameRoman;
        
        var result = await _userService.UpdateUserAsync(user);
        if (result)
        {
            return NoContent();
        }
        
        return BadRequest(new { message = "ユーザー更新に失敗しました" });
    }

    // Delete user
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(string id)
    {
        var user = await _userService.GetByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }
        
        var result = await _userService.DeleteUserAsync(id);
        if (result)
        {
            return NoContent();
        }
        
        return BadRequest(new { message = "ユーザー削除に失敗しました" });
    }

    // Assign role to user
    [HttpPost("{userId}/roles")]
    public async Task<IActionResult> AssignRole(string userId, [FromBody] AssignRoleDto model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        var user = await _userService.GetByIdAsync(userId);
        if (user == null)
        {
            return NotFound();
        }
        
        var result = await _userService.AddToRoleAsync(userId, model.RoleName);
        if (result)
        {
            return NoContent();
        }
        
        return BadRequest(new { message = "ロール割り当てに失敗しました" });
    }

    // Remove role from user
    [HttpDelete("{userId}/roles/{roleName}")]
    public async Task<IActionResult> RemoveRole(string userId, string roleName)
    {
        var user = await _userService.GetByIdAsync(userId);
        if (user == null)
        {
            return NotFound();
        }
        
        var result = await _userService.RemoveFromRoleAsync(userId, roleName);
        if (result)
        {
            return NoContent();
        }
        
        return BadRequest(new { message = "ロール解除に失敗しました" });
    }

    // Get user roles
    [HttpGet("{userId}/roles")]
    public async Task<IActionResult> GetUserRoles(string userId)
    {
        var user = await _userService.GetByIdAsync(userId);
        if (user == null)
        {
            return NotFound();
        }
        
        var roles = await _userService.GetUserRolesAsync(userId);
        return Ok(roles);
    }
}
```

## DTOモデル (バリデーション追加)

### 認証関連

```csharp
public class LoginDto
{
    [Required(ErrorMessage = "ログインIDは必須です")]
    public string LoginId { get; set; } = string.Empty;
    
    [Required(ErrorMessage = "パスワードは必須です")]
    public string Password { get; set; } = string.Empty;
    
    public bool RememberMe { get; set; } = false;
}

public class UserInfoDto
{
    public string Id { get; set; } = string.Empty;
    public string LoginId { get; set; } = string.Empty;
    public string UserName { get; set; } = string.Empty;
    public string? UserNameKana { get; set; }
    public string? UserNameRoman { get; set; }
    public List<string> Roles { get; set; } = new List<string>();
}
```

### ユーザー関連

```csharp
public class CreateUserDto
{
    [Required(ErrorMessage = "ログインIDは必須です")]
    [MinLength(4, ErrorMessage = "ログインIDは4文字以上必要です")]
    public string LoginId { get; set; } = string.Empty;
    
    [Required(ErrorMessage = "ユーザー名は必須です")]
    public string UserName { get; set; } = string.Empty;
    
    public string? UserNameKana { get; set; }
    
    public string? UserNameRoman { get; set; }
    
    [Required(ErrorMessage = "パスワードは必須です")]
    [MinLength(6, ErrorMessage = "パスワードは6文字以上必要です")]
    public string Password { get; set; } = string.Empty;
    
    public List<string> Roles { get; set; } = new List<string>();
}

public class UpdateUserDto
{
    public string? UserName { get; set; }
    public string? UserNameKana { get; set; }
    public string? UserNameRoman { get; set; }
    public string? Password { get; set; }
}

public class AssignRoleDto
{
    [Required(ErrorMessage = "ロール名は必須です")]
    public string RoleName { get; set; } = string.Empty;
}
```

## Identityセットアップ (Program.cs)

```csharp
// Identity設定
builder.Services.AddIdentity<UserAccount, Role>(options =>
{
    // パスワードポリシー
    options.Password.RequireDigit = true;
    options.Password.RequireLowercase = true;
    options.Password.RequireUppercase = false;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequiredLength = 6;
    
    // ユーザー名の検証
    options.User.RequireUniqueEmail = false;
    options.User.AllowedUserNameCharacters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._@+";
})
.AddEntityFrameworkStores<AppDbContext>()
.AddDefaultTokenProviders()
.AddUserStore<UserAccountStore>()
.AddRoleStore<RoleStore>();

// Cookie認証設定
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.ExpireTimeSpan = TimeSpan.FromHours(24);
    options.LoginPath = "/api/auth/login";
    options.LogoutPath = "/api/auth/logout";
    options.AccessDeniedPath = "/api/auth/access-denied";
    options.SlidingExpiration = true;
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});

// カスタムManagerを登録
builder.Services.AddScoped<UserAccountManager>();
builder.Services.AddScoped<ApplicationRoleManager>();

// サービス登録
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IRoleService, RoleService>();
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IRoleRepository, RoleRepository>();
```

## データベース設定 (AppDbContext)

```csharp
public class AppDbContext : IdentityDbContext<UserAccount, Role, string>
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        
        // UserAccountエンティティの設定
        builder.Entity<UserAccount>(entity =>
        {
            entity.ToTable("Users");
            entity.HasIndex(e => e.LoginId).IsUnique();
            
            // ロールとの関連付けを無視（Identityフレームワークが管理するため）
            entity.Ignore(e => e.Roles);
        });
        
        // Roleエンティティの設定
        builder.Entity<Role>(entity =>
        {
            entity.ToTable("Roles");
        });
        
        // その他のIdentityテーブル名のカスタマイズ
        builder.Entity<IdentityUserRole<string>>(entity =>
        {
            entity.ToTable("UserRoles");
        });
        
        builder.Entity<IdentityUserClaim<string>>(entity =>
        {
            entity.ToTable("UserClaims");
        });
        
        builder.Entity<IdentityUserLogin<string>>(entity =>
        {
            entity.ToTable("UserLogins");
        });
        
        builder.Entity<IdentityRoleClaim<string>>(entity =>
        {
            entity.ToTable("RoleClaims");
        });
        
        builder.Entity<IdentityUserToken<string>>(entity =>
        {
            entity.ToTable("UserTokens");
        });
    }
}
```

## プロジェクト構成

```
AuthGate/
├── AuthGate.API/                    # Web APIプロジェクト
│   ├── Controllers/                # コントローラ
│   ├── Program.cs                  # エントリーポイント
│   ├── appsettings.json            # 設定ファイル
│   └── Filters/                    # 認証・認可フィルター
├── AuthGate.Business/              # ビジネスロジック層
│   ├── Services/                   # サービスの実装
│   ├── Managers/                   # カスタムManager実装
│   └── DTOs/                       # データ転送オブジェクト
├── AuthGate.Data/                  # データアクセス層
│   ├── Repositories/               # リポジトリの実装
│   ├── Stores/                     # Identity用Store実装
│   ├── AppDbContext.cs             # DbContext
│   └── Migrations/                 # EF Core マイグレーション
└── AuthGate.Models/                # モデル層
    └── Entities/                   # エンティティクラス
```

## まとめ

このプロジェクトでは、LoginIDを使用したCookieベース認証のASP.NET Core Web APIを実装します。更新されたUserAccountモデルを基に、以下の構成を採用します：

1. **Store層** - Identity用のカスタムストア実装（UserAccountStore, RoleStore）
2. **Manager層** - Identity用のカスタムマネージャ実装（UserAccountManager, ApplicationRoleManager）
3. **Repository層** - データアクセス操作の簡略化
4. **Service層** - ビジネスロジックの実装
5. **Controller層** - RESTful APIエンドポイントの提供

標準のASP.NET Core Identityのバリデーション機能を最大限に活用し、DataAnnotationsを使用したモデル検証を実装します。これにより、拡張性が高く保守しやすいシステムを構築します。