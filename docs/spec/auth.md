## API 技術規格：

- 使用 Laravel Sanctum (laravel/sanctum) 進行 API 認證。
- 使用 Laravel RBAC (binary-cats/laravel-rbac) 進行角色和權限管理。

## API Endpoints:

- `POST /v1/auth/login`: 用戶登入，返回認證 token 。
- `POST /v1/auth/logout`: 用戶登出，撤銷認證 token 。 (需要 Bearer Token)
- `POST /v1/auth/register`: 用戶註冊，創建新用戶並返回認證 token 。
- `POST /v1/auth/refresh`: 刷新認證 token 。 (需要 Bearer Token)
- `POST /v1/auth/forgot-password`: 發送密碼重置郵件。
- `POST /v1/auth/reset-password`: 重置密碼。
- `POST /v1/auth/verify-email`: 驗證用戶電子郵件地址。 (需要 Bearer Token)
- `POST /v1/auth/resend-verification-email`: 重新發送驗證郵件。 (需要 Bearer Token)

## 權限、群組、角色：

- 免費用戶：
    - 可以使用 `/v1/auth/register` 註冊新帳號。
    - 可以使用 `/v1/auth/forgot-password` 發送密碼重置郵件。
    - 可以使用 `/v1/auth/reset-password` 重置密碼。
    - 可以使用 `/v1/auth/verify-email` 驗證電子郵件地址。
    - 可以使用 `/v1/auth/resend-verification-email` 重新發送驗證郵件。
    
- 付費用戶：
    - 可以使用 `/v1/auth/login` 登入。
    - 可以使用 `/v1/auth/logout` 登出。
    - 可以使用 `/v1/auth/refresh` 刷新認證 token 。
    - 可以建立群組，並自動將自己加入該群組，成為該群組的管理員。
    - 可以將在群組中建立子用戶。
