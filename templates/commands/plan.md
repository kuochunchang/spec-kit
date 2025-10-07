---
description: 使用計劃範本執行實作規劃工作流程，以生成設計工件。
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
---

使用者輸入可以由代理直接提供，也可以作為命令參數提供 - 在繼續執行提示之前，你**必須**考慮它（如果不為空）。

使用者輸入：

$ARGUMENTS

根據作為參數提供的實作細節，執行以下操作：

1. 從專案根目錄執行 `{SCRIPT}` 並解析 JSON 以獲取 FEATURE_SPEC、IMPL_PLAN、SPECS_DIR、BRANCH。所有未來的檔案路徑必須是絕對路徑。
   - 在繼續之前，檢查 FEATURE_SPEC 是否有 `## Clarifications` 區段，並至少有一個 `Session` 子標題。如果缺少或仍存在明顯的模糊區域（模糊的形容詞、未解決的關鍵選擇），請暫停並指示使用者先執行 `/clarify` 以減少返工。僅在以下情況下繼續：(a) 存在釐清說明 或 (b) 提供明確的使用者覆寫（例如，「在沒有釐清說明的情況下繼續」）。不要嘗試自己捏造釐清說明。
2. 閱讀並分析功能規格以了解：
   - 功能需求和使用者故事
   - 功能性和非功能性需求
   - 成功標準和驗收標準
   - 任何提及的技術限制或依賴項
3. 閱讀位於 `/memory/constitution.md` 的憲章以了解憲章要求。
4. 執行實作計劃範本：
   - 載入 `/templates/plan-template.md`（已複製到 IMPL_PLAN 路徑）
   - 將輸入路徑設定為 FEATURE_SPEC
   - 執行執行流程（主要）函數的步驟 1-9
   - 範本是獨立的且可執行的
   - 按照指定遵循錯誤處理和關卡檢查
   - 讓範本指導在 $SPECS_DIR 中生成工件：
     * 階段 0 生成 research.md
     * 階段 1 生成 data-model.md、contracts/、quickstart.md
     * 階段 2 生成 tasks.md
   - 將來自參數的使用者提供的細節整合到技術背景中：{ARGS}
   - 在完成每個階段時更新進度追蹤
5. 驗證執行完成：
   - 檢查進度追蹤顯示所有階段已完成
   - 確保所有必需的工件已生成
   - 確認執行中沒有 ERROR 狀態
6. 報告結果，包括分支名稱、檔案路徑和生成的工件。

對所有檔案操作使用帶有專案根目錄的絕對路徑，以避免路徑問題。
