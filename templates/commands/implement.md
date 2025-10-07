
---
description: 執行實作計畫，處理並執行 `tasks.md` 中定義的所有任務
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

使用者輸入可以由代理直接提供或作為命令參數 — 在繼續提示前你**必須**考慮該輸入（若非空）。

使用者輸入：

$ARGUMENTS

1. 從 repo 根目錄執行 `{SCRIPT}` 並解析 `FEATURE_DIR` 與 `AVAILABLE_DOCS` 清單。所有路徑必須為絕對路徑。

2. 載入並分析實作上下文：
   - **必要**：閱讀 `tasks.md` 以獲取完整任務清單與執行計畫
   - **必要**：閱讀 `plan.md` 以了解技術棧、架構與檔案結構
   - **若存在**：閱讀 `data-model.md` 以了解實體與關聯
   - **若存在**：閱讀 `contracts/` 以取得 API 規格與測試需求
   - **若存在**：閱讀 `research.md` 以了解技術決策與限制
   - **若存在**：閱讀 `quickstart.md` 以取得整合情境

3. 解析 `tasks.md` 結構並擷取：
   - **任務階段**：Setup、Tests、Core、Integration、Polish
   - **任務相依性**：順序執行或並行執行規則
   - **任務細節**：ID、描述、檔案路徑、並行標記 [P]
   - **執行流程**：順序與相依性需求

4. 根據任務計畫執行實作：
   - **分階段執行**：完成每個階段後再進入下一階段
   - **遵守相依性**：順序任務按序執行，標記為並行 [P] 的任務可同時執行  
   - **採用 TDD 流程**：先執行測試任務，再執行對應的實作任務
   - **檔案為基的協調**：影響相同檔案的任務必須順序執行
   - **驗證檢查點**：在繼續下一階段前驗證每個階段已完成

5. 實作執行規則：
   - **先 Setup**：初始化專案結構、相依套件與設定
   - **測試先於程式碼**：若需為 contracts、entities 或整合情境撰寫測試，先建立測試
   - **核心開發**：實作模型、服務、CLI 指令、端點
   - **整合工作**：資料庫連線、中介層、日誌、外部服務
   - **潤飾與驗證**：單元測試、效能優化、文件

6. 進度追蹤與錯誤處理：
   - 每完成一個任務就回報進度
   - 若任一非並行任務失敗，停止執行
   - 對於並行任務 [P]，繼續執行成功的任務並回報失敗的任務
   - 提供具上下文的清楚錯誤訊息以利除錯
   - 若無法繼續實作，建議下一步行動
   - **重要**：完成的任務請在 `tasks` 檔案中標記為 `[X]`。

7. 完成驗證：
   - 驗證所有必要任務已完成
   - 檢查已實作的功能是否符合原始規格
   - 驗證測試通過且覆蓋率符合需求
   - 確認實作遵循技術計畫
   - 回報最終狀態並摘要已完成的工作

注意：此命令假設 `tasks.md` 已有完整的任務拆解。如任務不完整或遺失，建議先執行 `/tasks` 以重新產生任務清單。

1. Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute.

2. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

3. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements

4. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together  
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

5. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

6. Progress tracking and error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **IMPORTANT** For completed tasks, make sure to mark the task off as [X] in the tasks file.

7. Completion validation:
   - Verify all required tasks are completed
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - Report final status with summary of completed work

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/tasks` first to regenerate the task list.