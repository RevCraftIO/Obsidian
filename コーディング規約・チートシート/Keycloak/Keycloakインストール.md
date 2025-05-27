---
作成日: 2025-05-27
tags:
  - Keycloak
  - 認証
  - 認可
---
# Keycloakインストール

## docker-compose.ymlファイルの準備
docker-compose.yml
ポートを`8080`から`8180`に変更しています。

```yaml:docker-compose.yml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:latest
    container_name: keycloak-postgres
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-keycloak}
      POSTGRES_USER: ${POSTGRES_USER:-keycloak}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-keycloak123}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - keycloak-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-keycloak}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Keycloak Server
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    container_name: keycloak-server
    environment:
      # Database Configuration
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB:-keycloak}
      KC_DB_USERNAME: ${POSTGRES_USER:-keycloak}
      KC_DB_PASSWORD: ${POSTGRES_PASSWORD:-keycloak123}
      
      # Keycloak Admin Configuration
      KEYCLOAK_ADMIN: ${KEYCLOAK_ADMIN:-admin}
      KEYCLOAK_ADMIN_PASSWORD: ${KEYCLOAK_ADMIN_PASSWORD:-admin123}
      
      # HTTP Configuration (no HTTPS required)
      KC_HTTP_ENABLED: "true"
      KC_HOSTNAME_STRICT: "false"
      KC_HOSTNAME_STRICT_HTTPS: "false"
      
      # Development/Production Mode
      KC_DEVELOPMENT: "false"
      
      # Logging
      KC_LOG_LEVEL: INFO
      
      # Health and Metrics
      KC_HEALTH_ENABLED: "true"
      KC_METRICS_ENABLED: "true"
    command:
      - start-dev
    volumes:
      - keycloak_data:/opt/keycloak/data
    ports:
      - "8180:8080"
    networks:
      - keycloak-network
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8080/health/ready || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s

volumes:
  postgres_data:
    driver: local
  keycloak_data:
    driver: local

networks:
  keycloak-network:
    driver: bridge  
```

```
# PostgreSQL Database Configuration
POSTGRES_DB=keycloak
POSTGRES_USER=keycloak
POSTGRES_PASSWORD=postgres_keycloak

# Keycloak Admin Configuration
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=admin_password

# Optional: Custom Realm Name (used in realm configuration)
REALM_NAME=company-realm
```
---
## コンテナの起動
```
docker compose up -d
```

## ボリュームの削除
作業を間違えた場合、以下のコマンドを実行して再度コンテナを起動すれば初期化できます。
```
docker compose down -v
```

---
## 次の手順
- [Keycloak初期設定](Keycloak初期設定.md)
