## フェーズ３：発展的な内容

8.  **役割ベースの認証（Role-Based Authorization）の実装**

    -  **役割モデルの作成:** `Models` フォルダーに `AppRole.cs` という新しいクラスを作成します。このクラスは `IdentityRole<Guid>` を継承します。
    -  **役割データモデルの作成:** 役割情報をJSONファイルに永続化するためのデータモデル `RoleData.cs` を `Models` フォルダーに作成します。これには `Id`、`Name`、`NormalizedName` などのプロパティを含めます。
	-  **役割サービスの作成:** 役割情報の読み書きを行う `RoleService.cs` を `Services` フォルダーに作成します。このサービスは、役割の作成、取得、更新、削除、JSONファイルへの保存と読み込みなどの機能を提供します。`appsettings.json` から役割情報JSONファイルのパスを読み込むようにします。
	-  **カスタムロールストアの実装:** `Stores` フォルダーに `RoleStore.cs` という新しいクラスを作成します。このクラスは `IRoleStore<AppRole>` インターフェースを実装します。コンストラクタで `RoleService` のインスタンスを依存性注入し、各インターフェースメソッド内で `RoleService` を使用して役割データの操作を行います（作成、削除、ID/名前での検索、名前の正規化など）。
	-  **Identityへのロールストアの登録:** `Program.cs` を開き、Identity サービス登録時にカスタムのロールストアを登録します。`builder.Services.AddRoleManager<RoleManager<AppRole>>().AddRoleStore<RoleStore>();` のように記述します。
	-  **初期役割の作成:** `Program.cs` または別の初期化クラスで、アプリケーション起動時に必要な初期役割（例: "Admin"）が存在しない場合に `RoleManager` を使用して作成し、`RoleService` を通じてJSONファイルに保存する処理を実装します。
	-  **ユーザーへの役割の割り当て:** `UserService` を修正し、ユーザーに役割を割り当てる機能を追加します。これには、ユーザーIDと役割名を関連付けてJSONファイルに保存する仕組みが必要です。新しいデータモデル（例: `UserRoleData.cs`）を作成し、それに対応する読み書き処理を `UserService` に実装します。`UserManager` の `AddToRoleAsync` を直接使用するのではなく、`UserService` 内でこれらの関連情報を管理します。
	-  **役割ベースの認可の実装:** APIコントローラーまたはアクションメソッドに `[Authorize(Roles = "Admin")]` 属性を追加することで、"Admin" ロールを持つ認証済みユーザーのみがアクセスできるようになります。複数の役割を許可する場合は `[Authorize(Roles = "Admin,Editor")]` のように指定します。

9.  **Identityのカスタマイズ**

	-  **カスタムユーザープロパティの追加:** `Models` フォルダーの `AppUser.cs` を開き、`FirstName`、`LastName`、`RegistrationDate` など、アプリケーションに必要なカスタムプロパティを追加します。
	-  **カスタムプロパティの永続化:** `Models` フォルダーの `UserData.cs` を開き、`AppUser.cs` に追加したカスタムプロパティに対応するプロパティを追加します。
	-  **データサービスの更新:** `UserService` のユーザー作成、取得、更新処理を修正し、JSONファイルへのカスタムプロパティの読み書き処理を実装します。ユーザー情報の保存
	-  **パスワードポリシーの設定:** `Program.cs` で `builder.Services.Configure<IdentityOptions>(options => { ... });` を使用して、パスワードの要件（最小長、大文字・小文字・数字・記号の必須など）を設定します。
	-  **ロックアウト設定:** 同じく `IdentityOptions` 内で、アカウントロックアウトのポリシー（失敗許容回数、ロックアウト時間など）を設定します。
	-  **エラーメッセージのカスタマイズ:** 必要に応じて、`IdentityErrorDescriber` クラスを継承するカスタムクラス（例: `CustomIdentityErrorDescriber.cs`）を作成し、エラーメッセージをアプリケーションに適した内容（例: 日本語化）にカスタマイズします。作成したカスタムエラー記述子を `Program.cs` で `builder.Services.AddIdentity<...>().AddUserStore<...>().AddErrorDescriber<CustomIdentityErrorDescriber>();` のように登録します。