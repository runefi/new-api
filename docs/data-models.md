# Data Models Inventory

## Scope

- Source directory: `model/`
- Inclusion rule: all `type XXX struct` definitions
- Grouping:
  - Persistent models: involved in `AutoMigrate` and used as database entities
  - Auxiliary models: query params, payloads, statistics, snapshots, and helper structures

## Persistent Models (AutoMigrate)

| Struct | File | Role | 中文说明 |
| --- | --- | --- | --- |
| Channel | `model/channel.go` | Upstream channel configuration and routing metadata | 上游渠道配置与调度路由元数据 |
| Token | `model/token.go` | API token quota, permissions, and status | API 令牌的额度、权限和状态信息 |
| User | `model/user.go` | User account, quota, auth bindings, and profile | 用户账号、额度、登录绑定与资料信息 |
| PasskeyCredential | `model/passkey.go` | WebAuthn credential binding for users | 用户 WebAuthn/Passkey 凭证绑定 |
| Option | `model/option.go` | Global key-value system options | 全局键值系统配置项 |
| Redemption | `model/redemption.go` | Redemption code lifecycle and usage records | 兑换码生命周期与使用记录 |
| Ability | `model/ability.go` | Group-model-channel capability mapping | 分组-模型-渠道能力映射关系 |
| Log | `model/log.go` | Consumption and operation logs | 消费与操作日志记录 |
| Midjourney | `model/midjourney.go` | Midjourney task persistence | Midjourney 任务持久化数据 |
| TopUp | `model/topup.go` | Top-up order and payment status | 充值订单与支付状态信息 |
| QuotaData | `model/usedata.go` | Aggregated usage/quota statistics | 聚合后的用量与额度统计 |
| Task | `model/task.go` | Unified async task lifecycle and billing context | 统一异步任务生命周期与计费上下文 |
| Model | `model/model_meta.go` | Model catalog metadata and endpoint bindings | 模型目录元数据与端点绑定关系 |
| Vendor | `model/vendor_meta.go` | Vendor metadata for model catalog | 模型供应商元数据 |
| PrefillGroup | `model/prefill_group.go` | Prompt/template prefill groups | 提示词/模板预填分组 |
| Setup | `model/setup.go` | Setup initialization state | 系统初始化状态记录 |
| TwoFA | `model/twofa.go` | Two-factor authentication settings and lock state | 双因子认证配置与锁定状态 |
| TwoFABackupCode | `model/twofa.go` | 2FA backup code usage records | 双因子备份码使用记录 |
| Checkin | `model/checkin.go` | User check-in reward records | 用户签到奖励记录 |
| SubscriptionOrder | `model/subscription.go` | Subscription payment orders | 订阅支付订单记录 |
| UserSubscription | `model/subscription.go` | Active user subscription balance/period | 用户有效订阅余额与周期数据 |
| SubscriptionPreConsumeRecord | `model/subscription.go` | Pre-consume records before task execution | 任务执行前的订阅预扣记录 |
| CustomOAuthProvider | `model/custom_oauth_provider.go` | Custom OAuth provider configuration | 自定义 OAuth 提供方配置 |
| UserOAuthBinding | `model/user_oauth_binding.go` | User-to-custom-OAuth provider binding | 用户与自定义 OAuth 提供方绑定关系 |
| SubscriptionPlan | `model/subscription.go` | Subscription plans and pricing rules | 订阅套餐与定价规则 |

## Auxiliary Models (Non-AutoMigrate Structs)

| Struct | File | Role | 中文说明 |
| --- | --- | --- | --- |
| AbilityWithChannel | `model/ability.go` | Joined projection for ability + channel type | 能力与渠道类型的联合查询投影 |
| ChannelInfo | `model/channel.go` | Embedded/decoded channel extra configuration | 渠道扩展配置的嵌入/解析结构 |
| CheckinRecord | `model/checkin.go` | Output projection for check-in history | 签到历史输出结构 |
| accessPolicyPayload | `model/custom_oauth_provider.go` | Access policy JSON payload in custom OAuth | 自定义 OAuth 访问策略 JSON 载荷 |
| accessConditionItem | `model/custom_oauth_provider.go` | Single condition item inside access policy | 访问策略中的单个条件项 |
| Stat | `model/log.go` | Aggregated quota/rpm/tpm metrics | 聚合后的 quota/rpm/tpm 指标结构 |
| RecordConsumeLogParams | `model/log.go` | Parameters for writing consume logs | 写入消费日志参数结构 |
| RecordTaskBillingLogParams | `model/log.go` | Parameters for writing task billing logs | 写入任务计费日志参数结构 |
| sqliteColumnDef | `model/main.go` | SQLite schema patch helper structure | SQLite 表结构补丁辅助结构 |
| TaskQueryParams | `model/midjourney.go` | Query conditions for Midjourney tasks | Midjourney 任务查询参数 |
| BoundChannel | `model/model_meta.go` | Channel projection bound to model metadata | 模型元数据绑定的渠道投影 |
| Pricing | `model/pricing.go` | Pricing projection structure for model prices | 模型价格投影结构 |
| PricingVendor | `model/pricing.go` | Vendor projection for pricing APIs | 定价接口中的供应商投影结构 |
| SubscriptionSummary | `model/subscription.go` | View model combining subscription + plan | 订阅与套餐聚合视图结构 |
| SubscriptionPreConsumeResult | `model/subscription.go` | Result object for pre-consume operation | 订阅预扣流程结果结构 |
| SubscriptionPlanInfo | `model/subscription.go` | Lightweight plan info projection | 轻量级套餐信息投影结构 |
| Properties | `model/task.go` | Serialized task properties payload | 任务属性序列化载荷 |
| TaskPrivateData | `model/task.go` | Serialized task private data payload | 任务私有数据序列化载荷 |
| TaskBillingContext | `model/task.go` | Runtime task billing context | 任务运行时计费上下文 |
| SyncTaskQueryParams | `model/task.go` | Conditions for task synchronization queries | 任务同步查询条件结构 |
| taskSnapshot | `model/task.go` | Internal task status snapshot for updates | 任务状态更新内部快照结构 |
| TaskQuotaUsage | `model/task.go` | Task quota usage metric container | 任务额度使用统计容器 |
| UserBase | `model/user_cache.go` | Cached lightweight user structure | 缓存中的轻量用户结构 |

## Notes

- This inventory is generated from current code definitions under `model/`.
- `AutoMigrate` source: `model/main.go` (`migrateDB` and `migrateDBFast`).
- Some auxiliary models are API projections or runtime payloads; they are not standalone database tables.
- Table schema design document with Chinese field descriptions: `docs/data-table-design.md`.
