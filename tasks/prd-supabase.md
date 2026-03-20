# PRD: 接入 Supabase 数据库

## Overview
将本地开发环境的数据库从 SQLite 切换为 Supabase（托管 PostgreSQL），通过直接 PostgreSQL 连接字符串方式接入，无需引入新 SDK。现有 GORM + PostgreSQL 代码路径直接复用，仅修改连接配置。新环境全新数据开始，生产环境独立配置，多环境共存。

## Goals
- 本地开发/测试环境使用 Supabase 作为数据库后端
- 通过环境变量管理连接字符串，不硬编码凭据
- 复用现有 PostgreSQL 兼容代码，最小化改动范围
- 提供清晰的多环境配置示例（开发 vs 生产）
- 保持 SQLite/MySQL/PostgreSQL 三数据库兼容性不被破坏

## Quality Gates

以下命令必须在每个 User Story 完成后通过：
- `go build ./...` - 确保编译无误
- `go test ./...` - 确保测试通过

## User Stories

### US-001: 添加 Supabase 连接字符串支持
As a developer, I want to configure a Supabase PostgreSQL connection via environment variables so that I can use Supabase as the database backend without code changes.

**Acceptance Criteria:**
- [ ] `SQL_DSN` 环境变量支持 Supabase 标准连接格式：`postgresql://postgres:[password]@db.[project-ref].supabase.co:5432/postgres`
- [ ] `SQL_DSN` 同时支持 Supabase 连接池格式（pgBouncer）：`postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres`
- [ ] 设置 `SQL_DSN` 后，系统自动识别为 PostgreSQL 模式（`common.UsingPostgreSQL = true`）
- [ ] 启动日志正确打印数据库类型为 PostgreSQL

### US-002: 验证 PostgreSQL 代码路径与 Supabase 兼容性
As a developer, I want to verify that all existing PostgreSQL-compatible code works correctly with Supabase so that I don't encounter runtime errors.

**Acceptance Criteria:**
- [ ] GORM AutoMigrate 在 Supabase 上成功执行，所有表正常创建
- [ ] `commonGroupCol`、`commonKeyCol` 等 PostgreSQL 专用列引用正常工作
- [ ] `commonTrueVal`/`commonFalseVal` 布尔值处理正确（PostgreSQL 使用 `true`/`false`）
- [ ] 基础 CRUD 操作（用户、渠道、令牌）在 Supabase 上正常运行
- [ ] 所有现有 `go test ./...` 测试通过

### US-003: 创建多环境配置示例文件
As a developer, I want clear environment configuration examples for different deployment scenarios so that I can quickly set up any environment.

**Acceptance Criteria:**
- [ ] 在项目根目录创建或更新 `.env.example`，包含 Supabase 连接字符串示例（密码占位符）
- [ ] 注释说明 Supabase 直连 vs 连接池两种格式的区别和适用场景
- [ ] 添加开发环境（Supabase）和生产环境（自托管 PostgreSQL）的示例配置
- [ ] `README` 或 `docs/` 中更新数据库配置说明，新增 Supabase 配置章节
- [ ] `.gitignore` 确认包含 `.env`（不提交真实凭据）

### US-004: 验证 Supabase SSL 连接配置
As a developer, I want to ensure SSL/TLS connections to Supabase work correctly so that the connection is secure and stable.

**Acceptance Criteria:**
- [ ] Supabase 要求 SSL，连接字符串中正确处理 `sslmode` 参数（默认 `require`）
- [ ] 如当前 GORM 配置不支持 SSL 参数透传，添加必要的 DSN 解析处理
- [ ] 连接成功建立，无 SSL 握手错误
- [ ] 连接池配置（最大连接数、空闲连接数）适配 Supabase 免费层限制（最多 60 连接）

## Functional Requirements
- FR-1: 系统必须通过 `SQL_DSN` 环境变量接受完整的 PostgreSQL 连接字符串
- FR-2: 当 `SQL_DSN` 包含 PostgreSQL 协议前缀时，自动设置 `UsingPostgreSQL = true`
- FR-3: 连接字符串中的 SSL 参数必须被正确透传给 GORM/database driver
- FR-4: 连接池参数必须可通过环境变量覆盖（`SQL_MAX_OPEN_CONNS`、`SQL_MAX_IDLE_CONNS`）
- FR-5: 不得破坏 SQLite 和 MySQL 的现有连接配置方式

## Non-Goals
- 不引入 Supabase Go SDK 或 Supabase REST API
- 不迁移现有数据库数据到 Supabase
- 不实现 Supabase 特有功能（Realtime、Storage、Auth）
- 不修改生产环境的数据库配置
- 不删除对 SQLite 或 MySQL 的支持

## Technical Considerations
- **现有入口**：数据库连接初始化在 `model/main.go`，重点关注此文件
- **环境变量**：检查 `common/` 目录下环境变量读取逻辑，确认 `SQL_DSN` 变量名
- **PostgreSQL 驱动**：项目已有 `gorm.io/driver/postgres`，无需新增依赖
- **Supabase 免费层连接限制**：默认最多 60 个直连，建议使用连接池（pgBouncer 端口 6543）
- **连接字符串格式**：Supabase 直连端口 5432，pgBouncer 端口 6543，两者略有差异

## Success Metrics
- `go build ./...` 零错误
- `go test ./...` 全部通过
- 使用 Supabase 连接字符串启动服务后，管理后台可正常登录和操作
- 数据库表通过 AutoMigrate 在 Supabase 控制台可见

## Open Questions
- 当前项目中 `SQL_DSN` 的确切环境变量名是否与假设一致？（需确认 `model/main.go` 或 `common/` 中的实际变量名）
- Supabase 项目是否已创建？是否需要在 PRD 中包含 Supabase 项目初始化步骤？