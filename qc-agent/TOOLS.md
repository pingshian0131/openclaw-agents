# Tools

所有正式資料存取只使用經核准的 QMS API。此檔只記連線方式與憑證引用，不保存秘密。

## QMS API

- **Base URL env:** `QMS_API_BASE_URL`
- **Credential env:** `QMS_API_TOKEN`
- **Credential reference:** `secret://freeforjoy/qc-agent/qms-api-token`
- **Tenant/Site env:** `QMS_SITE_ID`
- **Timeout env:** `QMS_API_TIMEOUT_SECONDS`

預期能力：

- 查詢規格、檢驗單、原始結果、批次狀態與稽核軌跡
- 查詢冷凍製程、冷庫、運輸溫度與 logger 資料
- 查詢 NCR、CAPA、留樣、客訴、追溯與召回資料
- 建立檢驗、複驗、NCR、CAPA、偏差調查與召回評估草稿

## 使用限制

- 禁止資料庫直連與任意 SQL。
- 禁止在 prompt、log、記憶或回覆中顯示 token、密碼或完整認證 header。
- 查詢優先使用唯讀 scope；寫入只允許建立草稿或核准後的受控命令。
- QA 放行、解除隔離、特採、報廢及正式召回端點必須由後端再次驗證授權、狀態與稽核資訊。
- 工具不存在或未設定時，明確回報「尚未連線」，不可捏造結果。

