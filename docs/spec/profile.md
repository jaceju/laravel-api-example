## API 技術規格：

- 使用 Laravel Sanctum (laravel/sanctum) 進行 API 認證。
- 使用 Laravel RBAC (binary-cats/laravel-rbac) 進行角色和權限管理。
- 使用 Laravel Module (nwidart/laravel-modules) 進行模組化開發。
  - 參考 `docs/guides/pages/04-1-modular-architecture.md` 的模組化架構。

## API Endpoints:

- `GET /v1/me`: 獲取當前認證用戶的詳細信息。 (需要 Bearer Token)
- `POST /v1/me`: 創建新的用戶個人資料。 (需要 Bearer Token)
- `PUT /v1/me`: 更新當前用戶的個人資料。 (需要 Bearer Token)
- `POST /v1/profiles`: 創建新的用戶個人資料。 (需要 Bearer Token)
- `GET /v1/profiles/{userId}`: 獲取特定用戶的個人資料。 (需要 Bearer Token)
- `PUT /v1/profiles/{userId}`: 更新特定用戶的個人資料。 (需要 Bearer Token)
- `DELETE /v1/profiles/{userId}`: 刪除特定用戶的個人資料。 (需要 Bearer Token)

## 權限、群組、角色：

- 付費用戶：
    - 可以使用 `/v1/me` 獲取自己的個人資料。
    - 可以使用 `/v1/me` 創建新的個人資料。
    - 可以使用 `/v1/me` 更新自己的個人資料。
    - 可以使用 `/v1/profiles/{userId}` 獲取子用戶的個人資料。 (需要群組管理員權限)
    - 可以使用 `/v1/profiles` 創建新的子用戶個人資料。 (需要群組管理員權限)
    - 可以使用 `/v1/profiles/{userId}` 更新子用戶的個人資料。 (需要群組管理員權限)
    - 可以使用 `/v1/profiles/{userId}` 刪除子用戶的個人資料。 (需要群組管理員權限)
