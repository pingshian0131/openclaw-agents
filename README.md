# openclaw-agents

三隻 [OpenClaw](https://docs.openclaw.ai/zh-TW) Agent 的 workspace 定義，以及一份可重複使用的 Agent 設計計畫書模板。

情境設定為食品製造工廠的營運管理 —— 人力資源、原物料與品質管理三個領域各由一隻 Agent 負責。這個 repo 的重點不在領域知識，而在 **一隻可以安全部署到真實營運環境的 Agent，它的規則檔應該怎麼寫**：權限邊界、人工核准紅線、記憶隔離、憑證處理。

## 目錄結構

```
hr-agent/          人力資源管理 Agent workspace
mc-agent/          原物料管理 Agent workspace
qc-agent/          品質管理 Agent workspace
claude_output/     設計文件與計畫書模板（HTML / DOCX）
```

每個 agent 目錄都是一個完整的 OpenClaw workspace，可以直接複製到 `~/.openclaw/workspace`（或多 Agent 部署時各自的 workspace 路徑）使用。

## 三隻 Agent

| Agent | | 職責範圍 | 權威資料來源 |
|---|---|---|---|
| [`hr-agent`](hr-agent/) | 👥 | 工作曆、輪班、加班申請與核准、出勤、補休、員工訓練與產線資格 | HRM API |
| [`mc-agent`](mc-agent/) | 📦 | 供應商、採購收貨、lot／效期、FEFO、庫存儲位、製令備料、批次追溯、冷鏈交接 | ERP／材料管理 API（品質狀態以 QMS 為準） |
| [`qc-agent`](qc-agent/) | 🧪 | 進料／製程／成品檢驗、冷鏈溫度偏差、待驗隔離放行狀態、NCR、CAPA、客訴與召回 | QMS API |

三隻都是 **查詢 + 分析 + 建立草稿** 的角色。庫存異動、品質放行、加班核准、薪資變更、對外通知一律不由 Agent 決定。

## Workspace 六檔案分工

每個 agent 目錄下的檔案不是任意切分的 —— 分工對應「哪些規則在哪些情境下還在生效」：

| 檔案 | 寫什麼 | 為什麼寫這裡 |
|---|---|---|
| `IDENTITY.md` | 名字、角色、emoji、一句話定位 | 最短的檔案，只放名牌資訊 |
| `SOUL.md` | 語氣、價值取捨、怎麼拒絕 | 人格層。要短 —— 位於 prompt cache 邊界之上，常改會讓快取失效 |
| `USER.md` | 使用者是誰、角色權限關係、語言時區 | 每 session 注入，有獨立字元預算，寫太多會被截斷 |
| `TOOLS.md` | API base URL、憑證來源、可用能力、使用限制 | 環境說明書。它不控制工具是否可用，只是給模型的指引 |
| `AGENTS.md` | 硬規則、權限、禁止事項、輸出格式 | **最重要的一份**。subagent session 只注入 `AGENTS.md`，想讓子代理遵守的規則只能寫這裡 |
| `MEMORY.md` | 跨 session 要記得的穩定事實與決策 | 精煉層，不是逐字稿 |
| `HEARTBEAT.md` | 定期巡檢設定 | 只在 heartbeat 事件時讀取。三隻目前都是 comments-only（預設停用） |
| `memory/` | `YYYY-MM-DD.md` 當日工作紀錄 | 工作層，可被 `memory_search` 檢索，不會每輪注入 |

同一條規則不重複寫進多個檔案 —— 重複的規則改版時只會被改到一份，剩下的變成互相矛盾的舊規則。規則的唯一家是 `AGENTS.md`。

## 共通設計原則

這三隻 Agent 反覆套用了幾個模式，是這個 repo 真正想示範的東西：

**憑證只寫佔位符** — `TOOLS.md` 一律寫 `${HRM_API_TOKEN}` 或 `secret://...` reference，真值留在環境變數／secret manager。所以這些檔案可以安心進 git。

**部署預設降級** — 在 `USER.md` 尚未填妥正式核准角色、`TOOLS.md` 尚未配置可驗證的 API scope 前，Agent 只能唯讀查詢與產生草稿，所有正式寫入能力保持停用。設定不完整時自動變安全，而不是自動變危險。

**人工核准紅線明確列舉** — 不寫「不越權」（願望），寫「不得核准自己建立的申請」（可檢查的規則）。

**記憶隔離** — 群組、共享或跨 Agent 情境不得載入 `MEMORY.md` 或 daily memory。這是防止記憶洩漏給不該看到的人的關鍵一行。

**記憶不能覆蓋規則** — 否則有人只要在對話裡說一次「以後這種都直接批准」，就可能被寫進記憶變成長期政策。

**不得繞道** — 「API 未提供的能力視為不可用，不得用 shell、瀏覽器或其他 Agent 繞過」。缺了這句，一隻拿不到 API 的 Agent 會很聰明地改用 exec 去翻檔案。

**區分事實層級** — 回覆明確標示 `已查證`／`系統計算`／`建議`／`待核准`／`未查證`，不把推測包裝成事實。

## claude_output/

設計過程的產出文件，可獨立閱讀：

| 檔案 | 內容 |
|---|---|
| `lobster_agent_plan_template_*.html` | 🦞 龍蝦計畫書 —— 設計一隻 OpenClaw Agent 的完整提案模板（8 個 Part + 交付檢查清單） |
| `龍蝦計畫書_填寫版_*.docx` | 同上的空白填寫版，可直接列印或發給團隊 |
| `openclaw_workspace_files_*.html` | Workspace 六個核心檔案的分工與寫法解析 |
| `openclaw_skills_*.html` | Skills（`SKILL.md`、frontmatter、載入優先序、`requires` gating）解析 |
| `openclaw_cron_*.html` | Automations／cron 排程參數與設計原則解析 |
| `markdown_format_guide_*.html` | Markdown 格式完全解析 |

計畫書的核心判準：Part 1（目的）與 Part 2（日常）決定後面全部。很多人一開始就在挑工具，結果做出一隻「什麼都能做、但沒有任何一件事是它負責」的 Agent。

## 使用方式

1. 複製其中一個 agent 目錄到你的 OpenClaw workspace 路徑。
2. 填妥 `USER.md` 的正式核准角色，以及 `TOOLS.md` 的 API endpoint 與 scope。
3. 以環境變數或 secret manager 注入憑證 —— **不要把真值寫回 workspace 檔案**。
4. 確認行為正確後，才考慮啟用 `HEARTBEAT.md` 或建立 cron job（建議新排程先 `--disabled`，手動 run 過再啟用）。

要設計自己的 Agent，從 `claude_output/` 的龍蝦計畫書開始，不要從複製 workspace 檔案開始。

## 注意事項

- 這些 workspace 定義的 API endpoint、scope 與核准角色都還是待設定狀態（各 `MEMORY.md` 的「待確認」一節有列）。它們是設計範本，不是即插即用的生產配置。
- `memory/*.md` 會累積實際營運資料（員工編號、批號、庫存數量）。目前 `.gitignore` 並未排除它們 —— 實際部署或 fork 後公開前，記得先加上 `*/memory/*.md`（並 `!*/memory/.gitkeep`），否則營運資料會被 commit 進去。
