---
description: 在任務產生之後，對 `spec.md`、`plan.md` 與 `tasks.md` 進行非破壞性的跨工件一致性與品質分析。
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

可以由代理直接提供或作為命令參數提供的使用者輸入 — 你在繼續提示前**必須**考慮它（如果不為空）。

使用者輸入：

$ARGUMENTS

目標：在實作前，辨識三個核心工件（`spec.md`, `plan.md`, `tasks.md`）之間的不一致、重複、模糊與規範不足之處。此命令**必須**僅在 `/tasks` 成功產出完整的 `tasks.md` 之後執行。

嚴格唯讀：請**勿**修改任何檔案。輸出一份結構化的分析報告。可選提供修正計畫（使用者必須明確同意，才會手動呼叫後續編輯命令）。

憲法權限：專案憲法（`/memory/constitution.md`）在此分析範圍內為**不可談判**。與憲法衝突的情況自動視為 CRITICAL，且需調整 spec、plan 或 tasks — 不可稀釋、重新解釋或默許忽略該原則。如果需要修改憲法本身，必須在 `/analyze` 之外另行以明確的憲法更新流程處理。

執行步驟：

1. 從 repo 根目錄執行 `{SCRIPT}` 一次，並解析 JSON 以取得 FEATURE_DIR 與 AVAILABLE_DOCS。推導絕對路徑：
   - SPEC = FEATURE_DIR/spec.md
   - PLAN = FEATURE_DIR/plan.md
   - TASKS = FEATURE_DIR/tasks.md
   如有任一必要檔案遺失則中止並回報錯誤訊息（指示使用者執行遺失的前置命令）。

2. 載入工件：
   - 解析 `spec.md` 的章節：Overview/Context、Functional Requirements、Non-Functional Requirements、User Stories、Edge Cases（若存在）。
   - 解析 `plan.md`：架構/技術棧選擇、資料模型參照、階段、技術限制。
   - 解析 `tasks.md`：任務 ID、描述、階段分組、並行標記 [P]、參考的檔案路徑。
   - 載入憲法 `/memory/constitution.md` 以進行原則驗證。

3. 建立內部語意模型：
   - 需求清單：每個功能性與非功能性需求都帶一個穩定金鑰（根據命令式片語推導 slug，例如 "User can upload file" -> `user-can-upload-file`）。
   - 使用者故事/行為清單。
   - 任務覆蓋對映：將每個任務映射到一或多個需求或故事（透過關鍵字或明示參照模式推斷，例如 ID 或關鍵片語）。
   - 憲法規則集：擷取原則名稱與任何 MUST/SHOULD 類的規範性敘述。

4. 偵測流程：
   A. 重複偵測：
      - 發現近似重複的需求。標記較差的措辭以便合併。
   B. 模糊偵測：
      - 標記缺乏可測量準則的模糊形容詞（fast、scalable、secure、intuitive、robust）。
      - 標記未解析的佔位符（TODO、TKTK、???、<placeholder> 等）。
   C. 規範不足：
      - 有動詞但缺少對象或可衡量結果的需求。
      - 缺乏驗收標準對齊的使用者故事。
      - 參考到 spec/plan 未定義的檔案或元件的任務。
   D. 與憲法對齊：
      - 任何與 MUST 原則衝突的需求或計劃元素。
      - 缺少憲法要求的段落或品質門檻。
   E. 覆蓋缺口：
      - 需求沒有任何對應任務。
      - 任務沒有對應到任何需求/故事。
      - 非功能性需求未反映在任務中（例如效能、安全）。
   F. 不一致：
      - 術語漂移（相同概念在不同檔案用不同名稱）。
      - 計劃中引用的資料實體在規格中缺失（或反之亦然）。
      - 任務排序矛盾（例如在沒有相依註記下，整合任務排在基礎設置任務之前）。
      - 相互衝突的需求（例如一處要求使用 Next.js，另一處要求使用 Vue）。

5. 嚴重度指派啟發式：
   - CRITICAL：違反憲法 MUST、缺少核心規格工件，或會阻塞基本功能的需求但沒有任何覆蓋。
   - HIGH：重複或衝突的需求、模糊的安全/效能屬性、不可測試的驗收標準。
   - MEDIUM：術語漂移、缺失非功能性任務覆蓋、規範不足的邊界情況。
   - LOW：措辭/風格改善、對執行順序影響小的輕微冗餘。

6. 產出一份 Markdown 報告（不寫檔）並包含章節：

   ### Specification Analysis Report
   | ID | Category | Severity | Location(s) | Summary | Recommendation |
   |----|----------|----------|-------------|---------|----------------|
   | A1 | Duplication | HIGH | spec.md:L120-134 | 兩個相似的需求 ... | 合併措辭；保留較清晰的版本 |
   （主表為每個發現新增一列；生成以類別首字母為前綴的穩定 ID。）

   額外子章節：
   - Coverage Summary Table：
     | Requirement Key | Has Task? | Task IDs | Notes |
   - Constitution Alignment Issues（如有）
   - Unmapped Tasks（如有）
   - Metrics：
     * Total Requirements
     * Total Tasks
     * Coverage % (至少有 1 個任務的需求比例)
     * Ambiguity Count
     * Duplication Count
     * Critical Issues Count

7. 在報告末尾輸出簡短的下一步動作區塊：
   - 若存在 CRITICAL 問題：建議在執行 `/implement` 之前先解決。
   - 若僅有 LOW/MEDIUM：使用者可繼續，但提供改進建議。
   - 提供明確的命令建議範例：例如 "執行 /specify 進行細化"、"執行 /plan 以調整架構"、"手動編輯 tasks.md 為 'performance-metrics' 新增覆蓋"。

8. 詢問使用者：「是否要我針對前 N 個問題建議具體的修正編輯？」（勿自動套用修改。）

行為規則：
- 絕對不要修改任何檔案。
- 不可憑空杜撰缺失章節 — 若不存在就直接回報。
- 保持結果可重複（若不做變更，重跑時應產生相同 ID 與計數）。
- 主表中的發現數量上限為 50；超出者在摘要中匯總說明。
- 若未發現任何問題，輸出成功報告並包含覆蓋統計與後續建議。

背景：{ARGS}
