# 数据表设计文档

## 1. 说明

- 数据来源：`model/` 目录中的 GORM 模型定义
- 覆盖范围：`migrateDB/migrateDBFast` 中参与 `AutoMigrate` 的持久化模型
- 字段说明规则：
  - “类型”列为 Go 字段类型
  - “约束/索引”列来自 `gorm` tag（简化展示）
  - “字段说明”为中文业务语义说明

## 2. 数据表清单

| 表名 | 结构体 | 文件 | 中文说明 |
| --- | --- | --- | --- |
| channels | Channel | model/channel.go | 渠道配置与路由策略表 |
| tokens | Token | model/token.go | API 访问令牌与额度控制表 |
| users | User | model/user.go | 用户账号与认证信息表 |
| passkey_credentials | PasskeyCredential | model/passkey.go | Passkey/WebAuthn 凭证表 |
| options | Option | model/option.go | 系统全局键值配置表 |
| redemptions | Redemption | model/redemption.go | 兑换码生成与使用记录表 |
| abilities | Ability | model/ability.go | 分组-模型-渠道能力映射表 |
| logs | Log | model/log.go | 请求消费与审计日志表 |
| midjourneys | Midjourney | model/midjourney.go | Midjourney 任务记录表 |
| top_ups | TopUp | model/topup.go | 充值订单流水表 |
| quota_data | QuotaData | model/usedata.go | 用户模型用量统计表 |
| tasks | Task | model/task.go | 通用异步任务状态表 |
| models | Model | model/model_meta.go | 模型元数据目录表 |
| vendors | Vendor | model/vendor_meta.go | 模型供应商元数据表 |
| prefill_groups | PrefillGroup | model/prefill_group.go | 预填模板分组表 |
| setups | Setup | model/setup.go | 系统初始化状态表 |
| two_fas | TwoFA | model/twofa.go | 用户双因子认证配置表 |
| two_fa_backup_codes | TwoFABackupCode | model/twofa.go | 双因子备份码记录表 |
| checkins | Checkin | model/checkin.go | 用户签到奖励记录表 |
| subscription_orders | SubscriptionOrder | model/subscription.go | 订阅支付订单表 |
| user_subscriptions | UserSubscription | model/subscription.go | 用户订阅额度与周期表 |
| subscription_pre_consume_records | SubscriptionPreConsumeRecord | model/subscription.go | 订阅预扣费记录表 |
| custom_oauth_providers | CustomOAuthProvider | model/custom_oauth_provider.go | 自定义 OAuth 提供方配置表 |
| user_oauth_bindings | UserOAuthBinding | model/user_oauth_binding.go | 用户 OAuth 绑定关系表 |
| subscription_plans | SubscriptionPlan | model/subscription.go | 订阅套餐配置表 |

## 3. 字段设计（含中文说明）

### 3.1 channels（渠道配置表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 渠道主键 ID |
| type | int | default:0 | 渠道类型（不同供应商） |
| key | string | not null | 渠道鉴权密钥 |
| name | string | index | 渠道名称 |
| status | int | default:1 | 渠道状态（启用/禁用） |
| group | string | default:'default' | 渠道所属分组 |
| models | string | - | 支持的模型列表 |
| base_url | *string | default:'' | 上游请求地址 |
| used_quota | int64 | default:0 | 累计已用额度 |
| created_time | int64 | bigint | 创建时间戳 |
| test_time | int64 | bigint | 最近测试时间 |

### 3.2 tokens（令牌表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 令牌主键 ID |
| user_id | int | index | 所属用户 ID |
| key | string | uniqueIndex | 访问令牌字符串 |
| name | string | index | 令牌名称 |
| status | int | default:1 | 令牌状态 |
| remain_quota | int | default:0 | 剩余额度 |
| unlimited_quota | bool | - | 是否无限额度 |
| model_limits_enabled | bool | - | 是否开启模型限制 |
| model_limits | string | text | 允许调用模型配置 |
| allow_ips | *string | default:'' | IP 白名单 |
| created_time | int64 | bigint | 创建时间戳 |
| accessed_time | int64 | bigint | 最近访问时间 |

### 3.3 users（用户表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 用户主键 ID |
| username | string | unique,index | 用户名 |
| password | string | not null | 密码哈希 |
| display_name | string | index | 显示名称 |
| email | string | index | 邮箱 |
| quota | int | default:0 | 账户总额度 |
| group | string | default:'default' | 用户分组 |
| aff_code | string | uniqueIndex | 邀请码 |
| inviter_id | int | index | 邀请人用户 ID |
| github_id | string | index | GitHub 绑定 ID |
| discord_id | string | index | Discord 绑定 ID |
| oidc_id | string | index | OIDC 绑定 ID |

### 3.4 passkey_credentials（Passkey 凭证表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | primaryKey | 凭证主键 ID |
| user_id | int | uniqueIndex,not null | 关联用户 ID |
| attestation_type | string | varchar(255) | 认证器证明类型 |
| sign_count | uint32 | default:0 | 签名计数器 |
| user_present | bool | - | 认证时用户在场标记 |
| user_verified | bool | - | 认证时用户验证标记 |
| transports | string | text | 凭证传输方式 |
| attachment | string | varchar(32) | 凭证附着类型 |
| last_used_at | *time.Time | - | 最近使用时间 |
| created_at | time.Time | - | 创建时间 |

### 3.5 options（系统配置表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| key | string | primaryKey | 配置键 |
| value | string | - | 配置值 |

### 3.6 redemptions（兑换码表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 兑换码记录 ID |
| key | string | uniqueIndex | 兑换码 |
| user_id | int | - | 创建者用户 ID |
| used_user_id | int | - | 实际使用者 ID |
| status | int | default:1 | 兑换码状态 |
| quota | int | default:100 | 可兑换额度 |
| created_time | int64 | bigint | 创建时间 |
| redeemed_time | int64 | bigint | 兑换时间 |

### 3.7 abilities（能力映射表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| group | string | primaryKey | 用户分组标识 |
| model | string | primaryKey | 模型名称 |
| channel_id | int | primaryKey,index | 绑定渠道 ID |
| enabled | bool | - | 映射是否启用 |
| priority | *int64 | index,default:0 | 调度优先级 |
| weight | uint | index,default:0 | 负载权重 |
| tag | *string | index | 渠道标签 |

### 3.8 logs（日志表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 联合索引 | 日志主键 ID |
| user_id | int | index | 用户 ID |
| created_at | int64 | bigint,index | 记录时间 |
| type | int | index | 日志类型 |
| model_name | string | index | 模型名称 |
| token_name | string | index | 令牌名称 |
| quota | int | default:0 | 消耗额度 |
| prompt_tokens | int | default:0 | 输入 token 数 |
| completion_tokens | int | default:0 | 输出 token 数 |
| channel_id | int | index | 渠道 ID |
| request_id | string | index | 请求追踪 ID |
| content | string | - | 日志内容 |

### 3.9 midjourneys（MJ 任务表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 任务主键 ID |
| user_id | int | index | 用户 ID |
| action | string | index | 任务动作类型 |
| mj_id | string | index | Midjourney 任务 ID |
| prompt | string | - | 原始提示词 |
| status | string | index | 任务状态 |
| progress | string | index | 任务进度 |
| submit_time | int64 | index | 提交时间 |
| start_time | int64 | index | 开始时间 |
| finish_time | int64 | index | 结束时间 |
| image_url | string | - | 结果图片地址 |
| quota | int | - | 任务消耗额度 |

### 3.10 top_ups（充值订单表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 充值订单 ID |
| user_id | int | index | 用户 ID |
| amount | int64 | - | 充值额度数量 |
| money | float64 | - | 支付金额 |
| trade_no | string | unique,index | 商户订单号 |
| payment_method | string | varchar(50) | 支付方式 |
| status | string | - | 订单状态 |
| create_time | int64 | - | 下单时间 |
| complete_time | int64 | - | 完成时间 |

### 3.11 quota_data（用量统计表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 统计记录 ID |
| user_id | int | index | 用户 ID |
| username | string | 联合索引 | 用户名快照 |
| model_name | string | 联合索引 | 模型名快照 |
| created_at | int64 | bigint,index | 统计时间 |
| token_used | int | default:0 | token 使用量 |
| count | int | default:0 | 请求次数 |
| quota | int | default:0 | 额度消耗值 |

### 3.12 tasks（异步任务表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int64 | primaryKey | 任务主键 ID |
| user_id | int | index | 用户 ID |
| channel_id | int | index | 渠道 ID |
| quota | int | - | 任务额度消耗 |
| submit_time | int64 | index | 提交时间 |
| start_time | int64 | index | 开始时间 |
| finish_time | int64 | index | 结束时间 |
| progress | string | index | 任务进度 |
| fail_reason | string | - | 失败原因 |
| properties | json | type:json | 任务公开属性 |
| private_data | json | type:json | 任务私有数据 |
| data | json.RawMessage | type:json | 上游任务结果数据 |

### 3.13 models（模型元数据表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 模型记录 ID |
| model_name | string | uniqueIndex | 模型名称 |
| vendor_id | int | index | 供应商 ID |
| description | string | text | 模型描述 |
| icon | string | varchar(128) | 模型图标 |
| tags | string | varchar(255) | 模型标签 |
| endpoints | string | text | 模型可用端点配置 |
| status | int | default:1 | 模型状态 |
| sync_official | int | default:1 | 是否同步官方模型 |
| name_rule | int | default:0 | 模型名匹配规则 |
| created_time | int64 | bigint | 创建时间 |
| updated_time | int64 | bigint | 更新时间 |

### 3.14 vendors（供应商表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 供应商 ID |
| name | string | uniqueIndex | 供应商名称 |
| description | string | text | 供应商说明 |
| icon | string | varchar(128) | 供应商图标 |
| status | int | default:1 | 供应商状态 |
| created_time | int64 | bigint | 创建时间 |
| updated_time | int64 | bigint | 更新时间 |

### 3.15 prefill_groups（预填组表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 预填组 ID |
| name | string | uniqueIndex | 预填组名称 |
| type | string | index | 预填组类型 |
| items | json | type:json | 预填项内容 |
| description | string | varchar(255) | 描述信息 |
| created_time | int64 | bigint | 创建时间 |
| updated_time | int64 | bigint | 更新时间 |

### 3.16 setups（安装初始化表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | uint | primaryKey | 初始化记录 ID |
| version | string | not null | 初始化版本号 |
| initialized_at | int64 | bigint,not null | 初始化时间戳 |

### 3.17 two_fas（双因子配置表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | primaryKey | 2FA 配置 ID |
| user_id | int | unique,index | 用户 ID（每用户一条） |
| is_enabled | bool | - | 是否启用 2FA |
| failed_attempts | int | default:0 | 连续失败次数 |
| locked_until | *time.Time | - | 锁定截止时间 |
| last_used_at | *time.Time | - | 最近验证时间 |
| created_at | time.Time | - | 创建时间 |
| updated_at | time.Time | - | 更新时间 |

### 3.18 two_fa_backup_codes（2FA 备份码表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | primaryKey | 备份码记录 ID |
| user_id | int | index | 用户 ID |
| is_used | bool | - | 是否已使用 |
| used_at | *time.Time | - | 使用时间 |
| created_at | time.Time | - | 创建时间 |

### 3.19 checkins（签到记录表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | primaryKey | 签到记录 ID |
| user_id | int | uniqueIndex | 用户 ID |
| checkin_date | string | uniqueIndex | 签到日期（YYYY-MM-DD） |
| quota_awarded | int | not null | 奖励额度 |
| created_at | int64 | bigint | 创建时间 |

### 3.20 subscription_orders（订阅订单表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 订阅订单 ID |
| user_id | int | index | 用户 ID |
| plan_id | int | index | 订阅计划 ID |
| money | float64 | - | 支付金额 |
| trade_no | string | unique,index | 交易订单号 |
| payment_method | string | varchar(50) | 支付渠道 |
| status | string | - | 订单状态 |
| provider_payload | string | text | 支付平台原始回调 |
| create_time | int64 | - | 创建时间 |
| complete_time | int64 | - | 完成时间 |

### 3.21 user_subscriptions（用户订阅表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 用户订阅记录 ID |
| user_id | int | index | 用户 ID |
| plan_id | int | index | 计划 ID |
| amount_total | int64 | bigint,default:0 | 总可用额度 |
| amount_used | int64 | bigint,default:0 | 已使用额度 |
| start_time | int64 | bigint | 生效开始时间 |
| end_time | int64 | bigint,index | 生效结束时间 |
| last_reset_time | int64 | bigint | 上次重置时间 |
| next_reset_time | int64 | bigint,index | 下次重置时间 |
| upgrade_group | string | default:'' | 升级后的用户组 |
| prev_user_group | string | default:'' | 升级前用户组 |

### 3.22 subscription_pre_consume_records（订阅预扣记录表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 预扣记录 ID |
| request_id | string | uniqueIndex | 请求追踪 ID |
| user_id | int | index | 用户 ID |
| user_subscription_id | int | index | 对应用户订阅 ID |
| pre_consumed | int64 | bigint,default:0 | 预扣额度 |
| created_at | int64 | bigint | 创建时间 |
| updated_at | int64 | bigint,index | 更新时间 |

### 3.23 custom_oauth_providers（自定义 OAuth 提供方表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | primaryKey | 提供方 ID |
| name | string | not null | 提供方显示名称 |
| slug | string | uniqueIndex,not null | 提供方唯一标识 |
| icon | string | default:'' | 图标名称 |
| enabled | bool | default:false | 是否启用 |
| client_id | string | varchar(256) | OAuth 客户端 ID |
| client_secret | string | varchar(512) | OAuth 客户端密钥 |
| authorization_endpoint | string | varchar(512) | 授权地址 |
| token_endpoint | string | varchar(512) | 令牌地址 |
| user_info_endpoint | string | varchar(512) | 用户信息地址 |
| scopes | string | default:'openid profile email' | 授权范围 |
| access_policy | string | text | 访问策略 JSON |
| created_at | time.Time | - | 创建时间 |
| updated_at | time.Time | - | 更新时间 |

### 3.24 user_oauth_bindings（用户 OAuth 绑定表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | primaryKey | 绑定记录 ID |
| user_id | int | uniqueIndex | 用户 ID |
| provider_id | int | uniqueIndex | 自定义 OAuth 提供方 ID |
| provider_user_id | string | uniqueIndex | 提供方侧用户 ID |
| created_at | time.Time | - | 创建时间 |

### 3.25 subscription_plans（订阅计划表）

| 字段 | 类型 | 约束/索引 | 字段说明 |
| --- | --- | --- | --- |
| id | int | 主键 | 订阅计划 ID |
| title | string | not null | 计划标题 |
| subtitle | string | default:'' | 计划副标题 |
| price_amount | float64 | decimal(10,6) | 价格金额 |
| currency | string | default:'USD' | 货币代码 |
| duration_unit | string | default:'month' | 周期单位 |
| duration_value | int | default:1 | 周期值 |
| custom_seconds | int64 | default:0 | 自定义周期秒数 |
| enabled | bool | default:true | 是否上架 |
| sort_order | int | default:0 | 排序值 |
| max_purchase_per_user | int | default:0 | 每用户最大购买次数 |
| total_amount | int64 | default:0 | 总额度 |
| quota_reset_period | string | default:'never' | 额度重置周期 |
| quota_reset_custom_seconds | int64 | default:0 | 自定义重置秒数 |
| created_at | int64 | bigint | 创建时间 |
| updated_at | int64 | bigint | 更新时间 |

## 4. 备注

- 表名按当前模型命名和 `TableName()` 显式定义整理。
- 复杂 JSON 字段（如 `properties`、`items`、`access_policy`）建议在业务层做 schema 校验。
- 若后续模型字段有变更，建议同步更新本设计文档。
