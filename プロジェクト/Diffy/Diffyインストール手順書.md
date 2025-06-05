---
作成日: 2025-06-05
tags:
  - AI
---
# Diffyインストール手順書

Difyの最新版をDocker Composeでインストールする手順は以下の通りです。

---

## ✅ 前提条件

- **Git**、**Docker**、**Docker Compose**がインストールされていること
- **Docker Desktop**を使用する場合、WSL2が有効化されていること（Windows環境）
- **仮想CPU 2個以上**、**メモリ8GB以上**を割り当てていること
- **PowerShell**を使用すること

---

## 🛠️ インストール手順

1. **Difyのソースコードをクローン**
```
git clone https://github.com/langgenius/dify.git cd dify/docker
```

2. **環境設定ファイルを作成**
```
cp .env.example .env
```

3. **Docker Composeでコンテナを起動**
```
docker compose up -d
```

4. **ブラウザで初期設定画面にアクセス**
```
http://localhost/install
```
