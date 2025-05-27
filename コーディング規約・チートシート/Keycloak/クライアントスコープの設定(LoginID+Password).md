---
作成日: 2025-05-27
tags:
  - Keycloak
  - 認証
  - 認可
---
## クライアントスコープの設定(LoginID+Password)

| スコープ名               | Assigned Type | プロトコル          | 説明                                                    |
| ------------------- | ------------- | -------------- | ----------------------------------------------------- |
| `acr`               | DEFAULT       | OpenID Connect | 認証コンテキスト情報（認証レベルの識別）をトークンに含める                         |
| `address`           | Optional      | OpenID Connect | ユーザーの住所情報（`street_address` など）                        |
| `basic`             | DEFAULT       | OpenID Connect | 基本的なクレーム（`sub`, `preferred_username` 等）               |
| `email`             | Optional      | OpenID Connect | メールアドレスとその検証状態（`email`, `email_verified`）             |
| `microprofile-jwt`  | Optional      | OpenID Connect | MicroProfile JWT向けのスコープ（Java EEベースのアプリ用）              |
| `offline_access`    | DEFAULT       | OpenID Connect | **リフレッシュトークン**を取得するために必須                              |
| `organization`      | Optional      | OpenID Connect | 組織に関するクレーム（用途不明瞭、Keycloakで未使用のことも多い）                  |
| `phone`             | Optional      | OpenID Connect | 電話番号関連のクレーム（`phone_number`, `phone_verified`）         |
| `profile`           | DEFAULT       | OpenID Connect | ユーザーの基本プロフィール（`name`, `family_name`, `given_name` など） |
| `role_list`         | DEFAULT       | SAML           | SAMLプロトコル向けのロール一覧スコープ                                 |
| `roles`             | DEFAULT       | OpenID Connect | **アクセストークンにユーザーのロール情報を含めるためのスコープ**                    |
| `saml_organization` | Optional      | SAML           | SAML向けの組織情報スコープ（通常未使用）                                |
| `service_account`   | None          | OpenID Connect | サービスアカウント専用のスコープ                                      |
| `web-origins`       | Optional      | OpenID Connect | 許可されたCORSのオリジン（`allowed_origins`）をトークンに含める            |


