# Pure API Project with Laravel

> 🚀 **純 API 專案** - 基於 Laravel 建構的現代化 RESTful API 服務

[![Laravel Version](https://img.shields.io/badge/Laravel-12.x-FF2D20.svg?style=flat&logo=laravel)](https://laravel.com)
[![PHP Version](https://img.shields.io/badge/PHP-8.2+-777BB4.svg?style=flat&logo=php)](https://php.net)
[![API Version](https://img.shields.io/badge/API-v1.0-00D8FF.svg?style=flat)](docs/v1/)
[![HATEOAS](https://img.shields.io/badge/HATEOAS-Supported-green.svg?style=flat)](docs/v1/openapi.yaml)

## 🎯 專案概述

這是一個**純 API 專案**，專注於提供高品質的 RESTful API 服務。本專案採用模組化架構，支援 HATEOAS 超媒體控制，使用 Laravel Sanctum 進行 API 認證。

### 🌟 核心特色

- **🔗 HATEOAS 支援** - 完整的超媒體作為應用程式狀態引擎
- **🔐 API Token 認證** - Laravel Sanctum 實現安全的 API 存取
- **🧩 模組化架構** - 使用 nWidart/laravel-modules 實現功能模組化
- **🔑 一次性密碼驗證** - Spatie Laravel One-Time Passwords 提供額外安全層
- **📊 強型別 Data Objects** - Spatie Laravel Data 提供資料驗證和轉換
- **🛡️ RBAC 權限控制** - Binary Cats Laravel RBAC 實現角色權限管理
- **🧪 現代化測試** - Pest 測試框架確保程式碼品質
- **📚 OpenAPI 文件** - 完整的 API 規範文件

## 🏗️ 技術架構

### 核心技術棧

| 技術組件 | 版本 | 用途 |
|---------|------|------|
| **Laravel** | 12.x | 核心 API 框架 |
| **Laravel Sanctum** | Latest | API Token 認證 |
| **nWidart/laravel-modules** | Latest | 模組化管理 |
| **spatie/laravel-one-time-passwords** | Latest | 一次性密碼驗證 |
| **spatie/laravel-data** | Latest | DTO 與資料驗證 |
| **binary-cats/laravel-rbac** | Latest | 權限管理系統 |
| **pestphp/pest** | Latest | 測試框架 |

### 🚫 明確排除的功能

作為純 API 專案，以下功能被明確排除：

- ❌ Blade 模板引擎
- ❌ Web 路由 (routes/web.php)
- ❌ Session 中間件
- ❌ CSRF 保護
- ❌ 前端資源編譯
- ❌ 視圖相關功能

## 🚀 快速開始

### 系統需求

- PHP 8.2+ (Laravel 12 支援 PHP 8.2 - 8.4)
- Composer 2.0+
- MySQL 8.0+ / PostgreSQL 13+
- Redis 6.0+ (推薦)

### 安裝步驟

```bash
# 1. 複製專案
git clone <repository-url>
cd <project-name>

# 2. 安裝相依套件
composer install

# 3. 環境設定
cp .env.example .env
php artisan key:generate

# 4. 資料庫設定
php artisan migrate
php artisan db:seed

# 5. 啟動開發伺服器
php artisan serve
```

## 🧪 測試

### 執行測試

```bash
# 執行所有測試
sail test -p

# 執行特定測試檔案
sail test --filter=AuthControllerTest

# 執行測試並產生覆蓋率報告
sail test --coverage
```

### 測試策略

- **Feature Tests** - API 端點功能測試
- **Unit Tests** - 業務邏輯單元測試
- **Integration Tests** - 模組間整合測試
- **HATEOAS Tests** - 超媒體連結驗證測試

## 📁 專案結構

```
├── app/
│   ├── Abilities/               # 權限能力定義 (Enums)
│   ├── Http/Controllers/        # API 控制器
│   ├── Http/Middleware/         # API 中間件 (權限檢查)
│   ├── Models/                  # Eloquent 模型
│   ├── Providers/               # 服務提供者
│   ├── Roles/                   # 角色定義 (RBAC)
│   └── Data/                    # 共用 Data Objects
├── modules/                     # 功能模組
│   └── Profile/                 # Profile 模組範例
│       ├── app/                 # 模組應用程式碼
│       │   ├── Abilities/       # 模組專用權限
│       │   ├── Data/            # 模組 Data Objects
│       │   └── Models/          # 模組模型
│       ├── config/              # 模組設定檔
│       ├── database/            # 模組資料庫
│       └── routes/              # 模組路由 (只有 api.php)
├── docs/                    # API 文件
│   └── v1/                  # API v1 文件
├── routes/
│   ├── api.php              # API 路由定義
│   └── console.php          # 指令路由
├── tests/                   # 測試檔案
│   ├── Feature/             # 功能測試
│   └── Unit/                # 單元測試
└── database/
    ├── migrations/          # 資料庫遷移
    ├── seeders/            # 資料填充
    └── factories/          # 模型工廠
```

## 🔧 開發指南

### 建立新模組

```bash
# 建立新模組
php artisan module:make ModuleName

# 為模組建立控制器
php artisan module:make-controller ApiController ModuleName

# 為模組建立模型
php artisan module:make-model ModelName ModuleName

# 為模組建立遷移
php artisan module:make-migration create_table_name ModuleName
```

### API 開發準則

1. **RESTful 設計** - 遵循 REST 架構風格
2. **HATEOAS 實現** - 所有回應包含導航連結
3. **統一回應格式** - 使用一致的 JSON 結構
4. **錯誤處理** - 提供清晰的錯誤訊息
5. **版本控制** - 透過 URL 路徑管理 API 版本

### 程式碼風格

遵循 PSR-12 編碼標準：

```bash
# 檢查及修正程式碼風格
./vendor/bin/pint
```

## 📚 相關文件

- [Laravel 12 特性與更新說明](docs/Laravel-12-Features.md)
- [API 權限架構設計指南](docs/API-Permission-Architecture.md)
- [模組化架構設計指南](docs/模組化架構設計指南.md)
- [API專案模組化實作範例](docs/API專案模組化實作範例.md)
- [Pure API 架構總結](docs/Pure-API-Architecture-Summary.md)
- [專案更新總結](docs/Project-Update-Summary.md)
- [OpenAPI 規範文件](docs/v1/)

## 🤝 貢獻指南

1. Fork 本專案
2. 建立功能分支 (`git checkout -b feature/amazing-feature`)
3. 提交變更 (`git commit -m 'Add some amazing feature'`)
4. 推送分支 (`git push origin feature/amazing-feature`)
5. 開啟 Pull Request

## 📄 授權條款

本專案採用 MIT 授權條款 - 詳見 [LICENSE](LICENSE) 檔案

## 📞 聯絡資訊

如有任何問題或建議，歡迎：

- 建立 Issue
- 提交 Pull Request
- 聯絡專案維護者

---

> 💡 **提示**: 這是一個純 API 專案，專注於提供穩定、高效能的 RESTful API 服務。所有前端交互都透過 API 端點進行。
