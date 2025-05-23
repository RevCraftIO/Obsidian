# フェーズ2：認証機能

## 目次
1. [認証関連のモデル実装](#認証関連のモデル実装)
2. [認証ストアの実装](#認証ストアの実装)
3. [認証コントローラーの実装](#認証コントローラーの実装)
4. [セキュリティ設定](#セキュリティ設定)

---

## 認証関連のモデル実装

### 1.1 ユーザーモデル

#### 基本ユーザー情報
| 項目 | 型 | 説明 |
|------|-----|------|
| Id | Guid | ユーザーID |
| UserName | string | ユーザー名（ログインID） |
| NormalizedUserName | string | 正規化されたユーザー名 |
| LastName | string | 姓 |
| FirstName | string | 名 |

#### セキュリティ関連情報
| 項目 | 型 | 説明 |
|------|-----|------|
| PasswordHash | string | パスワードハッシュ |
| SecurityStamp | string | セキュリティスタンプ |
| ConcurrencyStamp | string | 同時実行スタンプ |
| LockoutEnd | DateTimeOffset? | ロックアウト終了日時 |
| LockoutEnabled | bool | ロックアウト有効フラグ |
| AccessFailedCount | int | アクセス失敗回数 |

### 1.2 認証関連DTO

#### ログインリクエスト
| 項目 | 型 | 必須 | 説明 |
|------|-----|------|------|
| UserName | string | ○ | ユーザー名 |
| Password | string | ○ | パスワード |
| RememberMe | bool | - | ログイン状態の保持 |

#### 認証レスポンス
| 項目 | 型 | 説明 |
|------|-----|------|
| Token | string | JWTトークン |
| Expires | DateTime | トークン有効期限 |
| UserInfo | UserInfo | ユーザー情報 |

## 認証ストアの実装

### 2.1 ストアの特徴
- ファイルベースのJSONストレージ
- スレッドセーフな実装
- ユーザー情報の永続化
- パスワードハッシュの管理

### 2.2 主要機能
- ユーザーの作成・読み込み・更新・削除
- パスワードの検証
- ユーザー名の正規化
- ロックアウト管理

## 認証コントローラーの実装

### 3.1 ログインエンドポイント
- **メソッド**: POST
- **パス**: `/api/auth/login`
- **機能**:
  - ユーザー認証
  - ロックアウトチェック
  - JWTトークン生成
  - ユーザー情報返却

### 3.2 エラーハンドリング
- 無効な認証情報
- アカウントロック
- システムエラー

## セキュリティ設定

### 4.1 パスワードポリシー
| 設定項目 | 値 | 説明 |
|----------|-----|------|
| 最小長 | 8文字 | パスワードの最小長 |
| 大文字必須 | 有効 | 大文字を含める必要あり |
| 小文字必須 | 有効 | 小文字を含める必要あり |
| 数字必須 | 有効 | 数字を含める必要あり |
| 特殊文字必須 | 有効 | 特殊文字を含める必要あり |

### 4.2 ロックアウト設定
| 設定項目 | 値 | 説明 |
|----------|-----|------|
| 最大試行回数 | 5回 | ロックアウトまでの失敗回数 |
| ロックアウト時間 | 10分 | アカウントロックの継続時間 |

### 4.3 JWT設定
| 設定項目 | 値 | 説明 |
|----------|-----|------|
| 有効期限 | 480分 | トークンの有効期間 |
| 署名アルゴリズム | HMAC-SHA256 | トークンの署名方式 |

## 次のステップ
- [フェーズ3：ユーザー管理](phase3-user-management.md)に進む
- [メインドキュメント](../AuthAPI.md)に戻る 