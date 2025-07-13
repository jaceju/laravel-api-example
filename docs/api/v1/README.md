# OpenAPI 文件撰寫指南

本目錄包含了完整的 API 文件，使用 OpenAPI 3.0.3 規範編寫。本指南將幫助你理解如何撰寫和維護這些 API 文件。

## 📁 文件結構

```
docs/v1/
├── _common.yaml           # 共用組件、schemas 和錯誤響應
├── openapi-auth.yaml      # 身份驗證相關的 API 端點
├── openapi-profile.yaml   # 使用者檔案管理 API 端點
└── README.md             # 本撰寫指南
```

## 🎯 設計原則

### 1. 模組化架構
- **`_common.yaml`**: 包含所有共用組件，以 `_` 開頭表示這是內部使用的組件檔案
- **功能模組檔案**: 每個業務模組獨立一個檔案 (如 `openapi-auth.yaml`, `openapi-profile.yaml`)
- **引用機制**: 使用 `$ref` 引用共用組件，避免重複定義

### 2. 一致性原則
- 所有 API 都遵循相同的響應格式
- 統一的錯誤處理機制
- 一致的命名慣例和結構

### 3. HATEOAS 實現
- 每個響應都包含相關的超媒體連結
- 提供可執行的操作指引
- 幫助客戶端發現和導航 API

## 🛠️ 撰寫規範

### 檔案組織結構

#### _common.yaml (共用組件)
```yaml
openapi: 3.0.3
info:
  title: Laravel API - Common Components
  # ... 基本資訊
components:
  securitySchemes:    # 認證方式定義
  parameters:         # 共用參數 (分頁、排序等)
  schemas:           # 共用 schemas
    # 基礎響應格式
    SuccessResponse:
    PaginatedResponse:
    # 錯誤響應格式
    BaseError:
    ValidationError:
    AuthenticationError:
    # HATEOAS 連結
    ResourceLinks:
    HateoasLink:
  responses:         # 標準錯誤響應
    BadRequestError:
    UnauthorizedError:
    ValidationError:
    # ...
```

#### 功能模組檔案 (如 openapi-auth.yaml)
```yaml
openapi: 3.0.3
info:
  title: Laravel API - Authentication
  # ... 模組資訊
servers:
  - url: "{protocol}://{host}"
    # ... 服務器配置
paths:
  /v1/auth/register:    # 完整路徑，包含版本號
    post:
      # ... 端點定義
components:
  schemas:
    # 模組特定的 schemas
    User:
    RegisterRequest:
    LoginRequest:
```

### 關鍵撰寫規範

#### 1. 路徑定義
```yaml
paths:
  /v1/auth/login:           # ✅ 版本號在路徑中
    post:
      tags: [Authentication] # 用於文件分組
      operationId: loginUser  # 唯一操作 ID
      summary: "用戶登入"      # 簡短描述
      description: |          # 詳細描述，支援 Markdown
        執行用戶身份驗證...
```

#### 2. 響應格式標準化
```yaml
responses:
  '200':
    description: 登入成功
    content:
      application/json:
        schema:
          allOf:
            - $ref: './_common.yaml#/components/schemas/SuccessResponse'
            - type: object
              properties:
                data:
                  type: object
                  properties:
                    user:
                      $ref: '#/components/schemas/User'
                    token:
                      type: string
```

#### 3. 錯誤處理
```yaml
responses:
  '400':
    $ref: './_common.yaml#/components/responses/BadRequestError'
  '401':
    description: 認證失敗
    content:
      application/json:
        schema:
          $ref: './_common.yaml#/components/schemas/AuthenticationError'
        examples:
          invalid_credentials:
            summary: 無效認證資訊
            value:
              error:
                code: "INVALID_CREDENTIALS"
                message: "提供的認證資訊不正確"
                # ...
```

#### 4. HATEOAS 連結實現
```yaml
# 在 _common.yaml 中定義
ResourceLinks:
  type: object
  properties:
    self:
      $ref: '#/components/schemas/HateoasLink'
    related:
      type: object
      additionalProperties:
        $ref: '#/components/schemas/HateoasLink'
    actions:
      type: object
      additionalProperties:
        $ref: '#/components/schemas/HateoasLink'

# 在響應中使用
response_example:
  links:
    self:
      href: "/v1/auth/me"
      method: "GET"
    actions:
      logout:
        href: "/v1/auth/logout"
        method: "POST"
        description: "登出當前會話"
```

## 📝 撰寫最佳實踐

### 1. Schema 設計

#### 請求 Schema
```yaml
RegisterRequest:
  type: object
  required: [name, email, password, password_confirmation]
  properties:
    name:
      type: string
      minLength: 1
      maxLength: 255
      example: "John Doe"
      description: "用戶全名"
    email:
      type: string
      format: email
      maxLength: 255
      example: "john@example.com"
      description: "電子郵件地址 (必須唯一)"
```

#### 響應 Schema
```yaml
User:
  type: object
  properties:
    id:
      type: integer
      example: 1
      description: "唯一用戶識別碼"
    name:
      type: string
      example: "John Doe"
      description: "用戶全名"
    # 包含時間戳
    created_at:
      type: string
      format: date-time
      example: "2025-07-13T10:30:00.000000Z"
  required: [id, name, email, created_at, updated_at]
```

### 2. 範例 (Examples) 撰寫

#### 成功響應範例
```yaml
examples:
  successful_login:
    summary: "成功登入"
    description: "用戶提供正確認證資訊時的響應"
    value:
      message: "登入成功"
      data:
        user:
          id: 1
          name: "John Doe"
          email: "john@example.com"
        token: "1|abc123..."
        token_type: "Bearer"
      links:
        self:
          href: "/v1/auth/me"
          method: "GET"
        actions:
          logout:
            href: "/v1/auth/logout"
            method: "POST"
```

#### 錯誤響應範例
```yaml
examples:
  validation_failed:
    summary: "驗證失敗"
    value:
      error:
        code: "VALIDATION_FAILED"
        message: "提供的資料無效"
        details: "請檢查標記的欄位並重試"
        timestamp: "2025-07-13T10:30:00.000000Z"
        request_id: "req_abc123"
      data:
        errors:
          email: ["電子郵件欄位為必填", "電子郵件格式無效"]
          password: ["密碼至少需要 8 個字元"]
```

### 3. 分頁處理

#### 分頁參數定義
```yaml
# 在 _common.yaml 中
parameters:
  PageParameter:
    name: page
    in: query
    description: "頁碼 (從 1 開始)"
    required: false
    schema:
      type: integer
      minimum: 1
      default: 1
```

#### 分頁響應格式
```yaml
PaginatedResponse:
  allOf:
    - $ref: '#/components/schemas/SuccessResponse'
    - type: object
      properties:
        meta:
          $ref: '#/components/schemas/PaginationMeta'
        links:
          allOf:
            - $ref: '#/components/schemas/ResourceLinks'
            - type: object
              properties:
                first: { type: string, nullable: true }
                last: { type: string, nullable: true }
                prev: { type: string, nullable: true }
                next: { type: string, nullable: true }
```

### 4. 安全性定義

#### 認證方式
```yaml
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT  # 雖然使用 Sanctum，但保持格式一致
      description: |
        Laravel Sanctum API 令牌認證

        **格式:** `Bearer {sanctum_token}`

# 在端點中使用
security:
  - BearerAuth: []
```

## 🔄 維護工作流程

### 1. 新增 API 端點
1. 在對應的模組檔案中新增路徑定義
2. 定義請求和響應 schema
3. 提供詳細的範例
4. 確保錯誤處理完整
5. 更新相關的 HATEOAS 連結

### 2. 修改現有端點
1. 更新 schema 定義
2. 調整範例數據
3. 檢查向後相容性
4. 更新版本號 (如需要)

### 3. 共用組件管理
1. 新增共用 schema 到 `_common.yaml`
2. 在模組檔案中使用 `$ref` 引用
3. 保持命名一致性
4. 定期檢查未使用的組件

## 🧪 驗證工具

### 1. OpenAPI 驗證
```bash
# 使用 swagger-codegen 驗證
npx @apidevtools/swagger-cli validate openapi-auth.yaml

# 使用 spectral 進行進階驗證
npx @stoplight/spectral-cli lint openapi-auth.yaml
```

### 2. 文件預覽
```bash
# 使用 Swagger UI
npx swagger-ui-serve openapi-auth.yaml

# 使用 Redoc
npx redoc-cli serve openapi-auth.yaml
```

### 3. 代碼生成測試
```bash
# 生成客戶端 SDK 測試
npx @openapitools/openapi-generator-cli generate \
  -i openapi-auth.yaml \
  -g typescript-fetch \
  -o ./generated/client
```

## 📋 檢查清單

### 新端點檢查清單
- [ ] 路徑包含版本號 (`/v1/...`)
- [ ] 有適當的 `tags` 分組
- [ ] `operationId` 唯一且有意義
- [ ] 包含 `summary` 和 `description`
- [ ] 請求參數完整定義
- [ ] 響應格式符合標準
- [ ] 錯誤處理覆蓋完整
- [ ] 包含實際的範例數據
- [ ] HATEOAS 連結正確
- [ ] 安全性配置適當

### 文件品質檢查
- [ ] 無 OpenAPI 規範錯誤
- [ ] 所有 `$ref` 引用正確
- [ ] 範例數據與 schema 一致
- [ ] 描述清楚且有幫助
- [ ] 命名一致性
- [ ] 無未使用的組件

## 💡 進階技巧

### 1. 條件式 Schema
```yaml
# 根據操作類型調整 schema
ProfileRequest:
  type: object
  properties:
    first_name:
      type: string
    last_name:
      type: string
  # 創建時需要所有欄位
  required: [first_name, last_name]

ProfileUpdateRequest:
  allOf:
    - $ref: '#/components/schemas/ProfileRequest'
    - type: object
      # 更新時所有欄位都是可選的
      required: []
```

### 2. 多態響應
```yaml
# 使用 oneOf 處理不同類型的響應
ApiResponse:
  oneOf:
    - $ref: '#/components/schemas/SuccessResponse'
    - $ref: '#/components/schemas/ErrorResponse'
  discriminator:
    propertyName: type
```

### 3. 回呼 (Callbacks) 定義
```yaml
# 對於 webhook 或異步操作
callbacks:
  webhook:
    '{$request.body#/webhook_url}':
      post:
        requestBody:
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/WebhookPayload'
```

## 🔗 相關資源

- [OpenAPI 3.0.3 規範](https://spec.openapis.org/oas/v3.0.3)
- [Laravel Sanctum 文件](https://laravel.com/docs/sanctum)
- [HATEOAS 原則](https://restfulapi.net/hateoas/)
- [OpenAPI 最佳實踐](https://swagger.io/resources/articles/best-practices-in-api-design/)

---

## 更新日誌

### v1.0.0 (2025-07-13)
- 建立模組化 OpenAPI 文件架構
- 實現 Laravel Sanctum 認證文件
- 新增使用者檔案管理 API 文件
- 整合標準化錯誤處理機制
- 實現完整的 HATEOAS 支援
- 建立共用組件系統 (`_common.yaml`)

- **Base URL**: `http://localhost/v1` (開發環境)
- **認證方式**: Laravel Sanctum API Token (非 JWT)
- **回應格式**: JSON with HATEOAS
- **文件版本**: 1.0.0

### 3. 認證流程

```bash
# 1. 註冊新用戶
curl -X POST "http://localhost/v1/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'

# 回應包含 HATEOAS links:
# {
#   "message": "User registered successfully",
#   "data": { ... },
#   "links": {
#     "self": { "href": "http://localhost/v1/auth/me", "method": "GET" },
#     "actions": {
#       "logout": { "href": "http://localhost/v1/auth/logout", "method": "POST" },
#       "profiles": { "href": "http://localhost/v1/profiles", "method": "GET" }
#     }
#   }
# }

# 2. 登入取得 Sanctum API Token
curl -X POST "http://localhost/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "password123"
  }'

# 3. 使用 Sanctum Token 呼叫受保護的 API
curl -X GET "http://localhost/v1/auth/me" \
  -H "Authorization: Bearer YOUR_SANCTUM_TOKEN_HERE"
```

## 📖 API 端點概覽

### 認證端點 (Authentication)

| 方法 | 端點 | 描述 | 認證需求 |
|------|------|------|----------|
| POST | `/v1/auth/register` | 註冊新用戶 | ❌ |
| POST | `/v1/auth/login` | 用戶登入 | ❌ |
| POST | `/v1/auth/logout` | 用戶登出 | ✅ |
| GET | `/v1/auth/me` | 取得當前用戶資訊 | ✅ |
| POST | `/v1/auth/refresh` | 刷新 Token | ✅ |
| GET | `/v1/user` | 取得用戶資訊 (舊版) | ✅ |

### Profile 端點

| 方法 | 端點 | 描述 | 認證需求 |
|------|------|------|----------|
| GET | `/v1/profiles` | 取得 Profile 列表 | ✅ |
| POST | `/v1/profiles` | 建立 Profile | ✅ |
| GET | `/v1/profiles/{id}` | 取得指定 Profile | ✅ |
| PUT | `/v1/profiles/{id}` | 完整更新 Profile | ✅ |
| PATCH | `/v1/profiles/{id}` | 部分更新 Profile | ✅ |
| DELETE | `/v1/profiles/{id}` | 刪除 Profile | ✅ |

## 🔒 權限控制

### 角色說明
- **一般用戶**: 只能存取自己的資源
- **管理員**: 可以存取所有資源

### Profile 權限規則
- 一般用戶只能：
  - 查看自己的 Profile
  - 建立自己的 Profile
  - 更新自己的 Profile
  - 刪除自己的 Profile
- 管理員可以：
  - 查看所有用戶的 Profile
  - 更新任何用戶的 Profile
  - 刪除任何用戶的 Profile

## 📋 回應格式

### HATEOAS 支援
此 API 遵循 HATEOAS (Hypermedia as the Engine of Application State) 原則，所有回應都包含相關的連結資訊。

### 成功回應 (含 HATEOAS)
```json
{
  "message": "操作成功的描述",
  "data": {
    // 回應資料
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/resource",
      "method": "GET"
    },
    "related": {
      "user": {
        "href": "http://localhost/v1/auth/me",
        "method": "GET"
      }
    },
    "actions": {
      "update": {
        "href": "http://localhost/v1/resource/1",
        "method": "PUT",
        "description": "Update this resource"
      },
      "delete": {
        "href": "http://localhost/v1/resource/1",
        "method": "DELETE",
        "description": "Delete this resource"
      }
    }
  }
}
```

---

# API 錯誤回應格式規範

## 概述

本 API 採用符合 HATEOAS（Hypermedia as the Engine of Application State）原則的標準化錯誤回應格式。所有錯誤回應都提供一致的結構，包含診斷資訊、相關連結和可行的操作建議。

## 基本錯誤回應結構

```json
{
  "error": {
    "code": "MACHINE_READABLE_ERROR_CODE",
    "message": "Human-readable error message",
    "details": "Additional context or explanation",
    "timestamp": "2025-07-13T10:30:00.000000Z",
    "request_id": "req_abc123def456",
    "trace_id": "trace_xyz789"
  },
  "data": {
    // Error-specific data
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/resource",
      "method": "GET"
    },
    "help": {
      "href": "https://api.example.com/docs/errors/not-found",
      "description": "Documentation for handling this error"
    },
    "actions": {
      "retry": {
        "href": "http://localhost/v1/resource",
        "method": "GET",
        "delay": 60,
        "description": "Retry the request"
      }
    }
  },
  "meta": {
    // Additional metadata if needed
  }
}
```

## 常見錯誤類型與範例

### 1. 驗證錯誤 (422 Unprocessable Entity)

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The given data was invalid.",
    "details": "Please check the highlighted fields and try again.",
    "timestamp": "2025-07-13T10:30:00.000000Z",
    "request_id": "req_abc123def456"
  },
  "data": {
    "errors": {
      "email": [
        "The email field is required.",
        "The email must be a valid email address."
      ],
      "password": [
        "The password must be at least 8 characters."
      ]
    },
    "failed_rules": {
      "email": ["required", "email"],
      "password": ["min:8"]
    }
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/auth/register",
      "method": "POST"
    },
    "help": {
      "href": "https://api.example.com/docs/validation",
      "description": "Validation rules documentation"
    },
    "actions": {
      "retry": {
        "href": "http://localhost/v1/auth/register",
        "method": "POST",
        "description": "Retry with corrected data"
      }
    }
  }
}
```

### 2. 身份驗證錯誤 (401 Unauthorized)

```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "The provided authentication token is invalid.",
    "details": "Token format is incorrect or token has been corrupted.",
    "timestamp": "2025-07-13T10:30:00.000000Z",
    "request_id": "req_def456ghi789"
  },
  "data": {
    "reason": "invalid_token"
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/profiles",
      "method": "GET"
    },
    "actions": {
      "login": {
        "href": "http://localhost/v1/auth/login",
        "method": "POST",
        "description": "Login to obtain a new authentication token"
      }
    },
    "help": {
      "href": "https://api.example.com/docs/authentication",
      "description": "Authentication documentation"
    }
  }
}
```

### 3. 授權錯誤 (403 Forbidden)

```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You do not have permission to perform this action.",
    "details": "This action requires admin privileges.",
    "timestamp": "2025-07-13T10:30:00.000000Z",
    "request_id": "req_abc123def456"
  },
  "data": {
    "required_permissions": ["admin.access", "profiles.delete"],
    "user_permissions": ["profiles.view", "profiles.update"],
    "resource_type": "profile",
    "resource_id": "123"
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/profiles/123",
      "method": "DELETE"
    },
    "help": {
      "href": "https://api.example.com/docs/permissions",
      "description": "Permissions documentation"
    },
    "actions": {
      "contact_support": {
        "href": "mailto:support@example.com",
        "description": "Contact support to request additional permissions"
      }
    },
    "related": {
      "profile": {
        "href": "http://localhost/v1/profiles/123",
        "method": "GET",
        "description": "View the profile (allowed action)"
      }
    }
  }
}
```

### 4. 資源未找到 (404 Not Found)

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found.",
    "details": "Profile with ID 999 does not exist.",
    "timestamp": "2025-07-13T10:30:00.000000Z",
    "request_id": "req_abc123def456"
  },
  "data": {
    "resource_type": "profile",
    "resource_id": "999",
    "search_criteria": {
      "id": 999
    }
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/profiles/999",
      "method": "GET"
    },
    "help": {
      "href": "https://api.example.com/docs/resources/profiles",
      "description": "Profile API documentation"
    },
    "alternatives": {
      "list_profiles": {
        "href": "http://localhost/v1/profiles",
        "method": "GET",
        "description": "List all available profiles"
      },
      "create_profile": {
        "href": "http://localhost/v1/profiles",
        "method": "POST",
        "description": "Create a new profile"
      }
    }
  }
}
```

### 5. 速率限制錯誤 (429 Too Many Requests)

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "details": "You have exceeded the rate limit of 60 requests per hour.",
    "timestamp": "2025-07-13T10:30:00.000000Z",
    "request_id": "req_abc123def456"
  },
  "data": {
    "limit": 60,
    "remaining": 0,
    "reset_time": "2025-07-13T11:00:00.000000Z",
    "retry_after": 3600
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/profiles",
      "method": "GET"
    },
    "help": {
      "href": "https://api.example.com/docs/rate-limits",
      "description": "Rate limiting documentation"
    },
    "actions": {
      "retry": {
        "href": "http://localhost/v1/profiles",
        "method": "GET",
        "delay": 3600,
        "description": "Retry after rate limit resets"
      }
    }
  }
}
```

### 6. 伺服器錯誤 (500 Internal Server Error)

```json
{
  "error": {
    "code": "DATABASE_ERROR",
    "message": "An internal server error occurred.",
    "details": "Unable to connect to the database. Please try again later.",
    "timestamp": "2025-07-13T10:30:00.000000Z",
    "request_id": "req_abc123def456"
  },
  "data": {
    "error_type": "database_error",
    "service": "database",
    "recovery_suggestions": [
      "Please try again in a few minutes",
      "If the problem persists, contact support"
    ]
  },
  "links": {
    "self": {
      "href": "http://localhost/v1/profiles",
      "method": "GET"
    },
    "actions": {
      "retry": {
        "href": "http://localhost/v1/profiles",
        "method": "GET",
        "delay": 60,
        "description": "Retry the request after a short delay"
      },
      "contact_support": {
        "href": "mailto:support@example.com",
        "description": "Contact support if the issue persists"
      }
    }
  }
}
```

## HTTP 標頭

錯誤回應會包含相關的 HTTP 標頭：

### 通用標頭
- `X-Request-ID`: 唯一的請求識別碼，用於除錯
- `Content-Type`: `application/json`

### 身份驗證錯誤
- `WWW-Authenticate`: `Bearer`

### 速率限制錯誤
- `X-RateLimit-Limit`: 每小時請求限制
- `X-RateLimit-Remaining`: 目前視窗剩餘請求數
- `X-RateLimit-Reset`: 速率限制重置的 Unix 時間戳
- `Retry-After`: 重試前等待的秒數

### 服務不可用錯誤
- `Retry-After`: 重試前等待的秒數

## HATEOAS 連結類型

### `self`
指向導致錯誤的原始請求

### `help`
指向相關文件或說明頁面

### `actions`
可以執行的操作：
- `retry`: 重試請求
- `login`: 登入以獲得權限
- `contact_support`: 聯絡支援
- `alternatives`: 替代操作

### `related`
相關資源的連結

## 錯誤碼對照表

| HTTP 狀態碼 | 錯誤碼 | 說明 |
|------------|--------|------|
| 400 | `BAD_REQUEST` | 請求格式錯誤 |
| 400 | `MALFORMED_JSON` | JSON 格式錯誤 |
| 400 | `INVALID_CONTENT_TYPE` | 不支援的內容類型 |
| 401 | `MISSING_TOKEN` | 缺少身份驗證令牌 |
| 401 | `INVALID_TOKEN` | 無效的身份驗證令牌 |
| 403 | `INSUFFICIENT_PERMISSIONS` | 權限不足 |
| 404 | `RESOURCE_NOT_FOUND` | 資源不存在 |
| 422 | `VALIDATION_FAILED` | 資料驗證失敗 |
| 429 | `RATE_LIMIT_EXCEEDED` | 超過速率限制 |
| 500 | `DATABASE_ERROR` | 資料庫錯誤 |
| 500 | `INTERNAL_SERVER_ERROR` | 伺服器內部錯誤 |
| 503 | `SERVICE_UNAVAILABLE` | 服務暫時不可用 |

---

## 📝 文件更新日誌

### v1.0.0 (2025-07-12)
- 初始版本
- 新增認證 API 文件 (Laravel Sanctum API Token)
- 新增 Profile API 文件
- 建立完整的 OpenAPI 規範
- 支援 HATEOAS (Hypermedia as the Engine of Application State)
- 所有回應包含相關連結和可用操作指引
