# フェーズ1：基本設定

## 目次
1. [プロジェクトの作成](#プロジェクトの作成)
2. [パッケージのインストール](#パッケージのインストール)
3. [基本設定クラスの実装](#基本設定クラスの実装)
4. [依存性注入の設定](#依存性注入の設定)

---

## プロジェクトの作成

### 1.1 ソリューション構成
- ソリューション名: `AuthAPI`
- プロジェクト構成:
  - `AuthApi`: メインのWeb APIプロジェクト
  - `AuthApi.Shared`: 共通ライブラリ
  - `AuthApi.Tests`: テストプロジェクト

### 1.2 プロジェクト構造
```
AuthAPI/
├── src/
│   ├── AuthApi/                # メインプロジェクト
│   │   ├── Controllers/       # APIコントローラー
│   │   ├── Models/           # データモデル
│   │   ├── Services/         # ビジネスロジック
│   │   └── Program.cs        # アプリケーションエントリーポイント
│   ├── AuthApi.Shared/        # 共通ライブラリ
│   │   ├── DTOs/             # データ転送オブジェクト
│   │   ├── Interfaces/       # インターフェース
│   │   └── Utils/            # ユーティリティ
│   └── AuthApi.Tests/         # テストプロジェクト
│       ├── Unit/             # 単体テスト
│       └── Integration/      # 統合テスト
├── docs/                      # ドキュメント
└── scripts/                   # デプロイメントスクリプト
```

## パッケージのインストール

### 2.1 メインプロジェクト（AuthApi）
| パッケージ | バージョン | 用途 |
|------------|------------|------|
| Microsoft.AspNetCore.Identity | 8.0.0 | 認証・認可機能 |
| Microsoft.Extensions.Options | 8.0.0 | 設定管理 |
| System.IdentityModel.Tokens.Jwt | 7.0.0 | JWTトークン処理 |

### 2.2 共通ライブラリ（AuthApi.Shared）
| パッケージ | バージョン | 用途 |
|------------|------------|------|
| Microsoft.Extensions.Options | 8.0.0 | 設定管理 |

### 2.3 テストプロジェクト（AuthApi.Tests）
| パッケージ | バージョン | 用途 |
|------------|------------|------|
| Moq | 4.20.0 | モック作成 |
| FluentAssertions | 6.12.0 | アサーション |

## 基本設定クラスの実装

### 3.1 アプリケーション設定

#### 基本設定
| 設定項目 | 型 | 説明 |
|----------|-----|------|
| UsersFilePath | string | ユーザーデータファイルのパス |
| AuditLogPath | string | 監査ログファイルのパス |

#### JWT設定
| 設定項目 | 型 | 説明 |
|----------|-----|------|
| SecretKey | string | トークン署名用の秘密鍵 |
| ExpirationMinutes | int | トークンの有効期限（分） |

#### セキュリティ設定
| 設定項目 | 型 | 説明 |
|----------|-----|------|
| MaxLoginAttempts | int | 最大ログイン試行回数 |
| LockoutMinutes | int | ロックアウト時間（分） |

### 3.2 設定ファイル構造
```json
{
  "AppSettings": {
    "UsersFilePath": "Data/users.json",
    "AuditLogPath": "Data/audit.log",
    "Jwt": {
      "SecretKey": "your-secret-key-here",
      "ExpirationMinutes": 480
    },
    "Security": {
      "MaxLoginAttempts": 5,
      "LockoutMinutes": 10
    }
  }
}
```

## 依存性注入の設定

### 4.1 サービス登録

#### 認証サービス
- JWT認証の設定
- トークン検証パラメータの設定
- 認証スキームの設定

#### ユーザーストア
- カスタムユーザーストアの登録
- ユーザーマネージャーの設定

### 4.2 ミドルウェア設定
- 認証ミドルウェア
- 認可ミドルウェア
- エラーハンドリング
- CORS設定

## 次のステップ
- [フェーズ2：認証機能](phase2-authentication.md)に進む
- [メインドキュメント](../AuthAPI.md)に戻る 