# Ralph Progress Log

This file tracks progress across iterations. Agents update this file
after each iteration and it's included in prompts for context.

## Codebase Patterns (Study These First)

### DB 类型检测模式（model/main.go `chooseDB`）
- `postgres://` 或 `postgresql://` 前缀 → PostgreSQL（设置 `common.UsingPostgreSQL = true`）
- `local` 前缀 → SQLite
- 其他 → MySQL（会自动追加 `parseTime=true`）
- Supabase 两种格式（标准 5432 端口、pgBouncer 6543 端口）均以 `postgresql://` 开头，无需特殊处理
- GORM postgres 配置使用 `PreferSimpleProtocol: true`（禁用隐式预编译语句，兼容 pgBouncer）

---

## 2026-03-20 - US-001
- 验证了 Supabase PostgreSQL 连接字符串支持是否已实现
- **结论：功能已存在，无需修改代码**
- 文件检查：`model/main.go`（`chooseDB` 函数）、`common/database.go`

**检查的文件：**
- `model/main.go:118-175`：`chooseDB` 函数已处理 `postgres://` 和 `postgresql://` 前缀
- `common/database.go`：定义 `UsingPostgreSQL`、`UsingSQLite`、`UsingMySQL` 标志

**Learnings：**
- Supabase 标准格式（`postgresql://postgres:[pwd]@db.[ref].supabase.co:5432/postgres`）和 pgBouncer 格式（`postgresql://postgres.[ref]:[pwd]@aws-0-[region].pooler.supabase.com:6543/postgres`）均以 `postgresql://` 开头，现有代码的 `strings.HasPrefix(dsn, "postgresql://")` 检测已完全覆盖
- `go build ./...` 在缺少 `web/dist`（前端未构建）时会失败，这是预存在的问题，与 Supabase 支持无关；所有后端包均可正常编译和测试
- GORM postgres 驱动使用 `PreferSimpleProtocol: true` 配合 pgBouncer 事务模式，保证兼容性
---
