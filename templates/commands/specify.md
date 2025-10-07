---
description: 從自然語言功能描述建立或更新功能規格。
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

您可以直接由代理提供輸入,或作為命令參數 - 您**必須**在繼續執行提示之前考慮它(如果不為空)。

使用者輸入:

$ARGUMENTS

使用者在觸發訊息中在 `/specify` 之後輸入的文字**就是**功能描述。假設您在此對話中始終可以使用它,即使下面字面上出現 `{ARGS}`。除非使用者提供空命令,否則不要要求使用者重複。

根據該功能描述,執行以下操作:

1. 從儲存庫根目錄執行腳本 `{SCRIPT}` 並解析其 JSON 輸出以獲取 BRANCH_NAME 和 SPEC_FILE。所有檔案路徑必須是絕對路徑。
  **重要** 您只能執行此腳本一次。JSON 作為輸出在終端中提供 - 始終參考它以獲取您正在尋找的實際內容。
2. 載入 `templates/spec-template.md` 以了解所需的章節。
3. 使用模板結構將規格寫入 SPEC_FILE,用從功能描述(參數)衍生的具體細節替換佔位符,同時保留章節順序和標題。
4. 報告完成情況,包含分支名稱、規格檔案路徑,以及準備進入下一階段。

注意:腳本會建立並切換到新分支,並在寫入前初始化規格檔案。
