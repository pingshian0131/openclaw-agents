# Tools

## HRM API

- **Base URL:** `${HRM_API_BASE_URL}`
- **Credential reference:** `${HRM_API_TOKEN}` 或執行環境的 secret manager reference
- **Tenant / factory:** `${HRM_TENANT_ID}`
- **Request actor:** `${OPENCLAW_ACTOR_ID}`
- **Correlation ID:** 每次跨系統請求產生並傳遞，用於 audit trace

預期能力：

- 查詢工作曆、班表、出勤日結與打卡紀錄
- 查詢加班、請假、補休申請及核准狀態
- 查詢員工職務、技能、訓練、證照與產線資格
- 建立排班、加班、補休或資格異動草稿／送審

## 使用限制

- 只呼叫已核准的 HRM API；禁止直連資料庫或使用共享管理員帳號。
- 認證資訊只能由環境變數或 secret manager 注入，不得寫入 workspace。
- 所有寫入都要附 actor、reason、correlation ID，並保留 API 回傳的 audit ID。
- API 未提供的能力視為不可用，不得用 shell、瀏覽器或其他 Agent 繞過。
- 回覆不得顯示 authorization header、token、cookie 或完整原始 PII payload。

## 尚待設定

- HRM OpenAPI / API 文件位置：`${HRM_API_DOCS_URL}`
- 允許的 OAuth scopes：`${HRM_API_SCOPES}`
- 核准角色與 escalation contact：`${HRM_APPROVER_REFERENCE}`
