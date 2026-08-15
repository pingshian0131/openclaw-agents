# TOOLS.md — mc-agent

只使用核准的 ERP／材料管理與 QMS API。不得直接連線資料庫，不得把秘密寫入本 workspace。

## ERP／材料管理 API

- **Base URL:** `${MC_API_BASE_URL}`
- **Credential reference:** `${MC_API_CREDENTIAL_REF}`
- **Service account:** `${MC_API_SERVICE_ACCOUNT}`
- **Tenant / Factory:** `${MC_API_TENANT_ID}` / `${MC_API_FACTORY_ID}`
- **Allowed scopes:** `supplier:read`, `purchase:read`, `receiving:read`, `material:read`, `inventory:read`, `work-order:read`, `traceability:read`, `cold-chain:read`, `draft:write`

預期能力：

- 查供應商、採購單、到貨與收貨。
- 查原料／添加物／包材 lot、效期、狀態、數量與儲位。
- 查製令需求、備料、投料及成品批次關聯。
- 查冷凍庫、運輸、logger 與冷鏈交接紀錄。
- 僅在 API 支援且使用者具權限時建立草稿；不得用通用寫入 endpoint 規避核准流程。

## QMS API（唯讀）

- **Base URL:** `${QMS_API_BASE_URL}`
- **Credential reference:** `${QMS_API_CREDENTIAL_REF}`
- **Service account:** `${QMS_API_SERVICE_ACCOUNT}`
- **Allowed scopes:** `inspection:read`, `lot-status:read`, `release:read`, `deviation:read`

用於確認待驗、隔離、放行、檢驗與冷鏈偏差狀態。mc-agent 不得呼叫品質放行或狀態覆寫工具。

## 工具使用規則

- credential reference 由部署環境的 secret manager 解析；不可在回覆、log 或 Markdown 中顯示秘密值。
- 所有 API 呼叫必須保留 request/correlation ID 與事件時間。
- 寫入僅限建立草稿；正式異動必須走來源系統的授權與簽核。
- API 失敗、資料過期或來源矛盾時停止推進，回報錯誤與待確認項目。
