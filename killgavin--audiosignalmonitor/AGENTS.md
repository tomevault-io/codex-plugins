<!-- 使用此檔案提供專案工作區的 Copilot 自訂指示。詳情請參閱 https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
- [x] 確認 .github 目錄下的 copilot-instructions.md 已建立。

- [x] 釐清專案需求
	<!-- 若未指定專案類型、語言或框架，請先詢問。若已提供則略過。 -->

- [x] 建立專案骨架
	<!--
	確認前一步已完成並標示完成。
	呼叫專案建立工具並帶入 projectType 參數。
	執行 scaffolding 命令建立專案檔案與資料夾。
	使用 '.' 作為工作目錄。
	若沒有合適的 projectType，請用工具搜尋文件。
	否則請使用檔案建立工具手動建立結構。
	-->

- [x] 客製化專案
	<!--
	確認前面步驟已完成並標示完成。
	依使用者需求制定修改計畫。
	使用適當工具與使用者提供的參考進行修改。
	若為 "Hello World" 專案可略過此步驟。
	-->

- [x] 安裝必要擴充套件
	<!-- 只安裝 get_project_setup_info 內指定的擴充套件，否則略過並標示完成。 -->

- [x] 編譯專案
	<!--
	確認前面步驟已完成。
	安裝缺少的相依套件。
	執行診斷並修正問題。
	檢查專案目錄內的 Markdown 檔案是否有相關指引。
	-->

- [x] 建立並執行工作
	<!--
	確認前面步驟已完成。
	參考 https://code.visualstudio.com/docs/debugtest/tasks 判斷是否需要工作。
	若需要，使用 create_and_run_task 依 package.json、README.md 與專案結構建立並啟動工作。
	否則略過。
	 -->

- [ ] 啟動專案
	<!--
	確認前面步驟已完成。
	詢問使用者是否以偵錯模式啟動，僅在確認後啟動。
	 -->

- [ ] 確保文件完整
	<!--
	確認前面步驟已完成。
	確認 README.md 與 .github 目錄下的 copilot-instructions.md 存在且內容為最新。
	移除 copilot-instructions.md 內所有 HTML 註解。
	 -->

<!--
## 執行指引
進度追蹤：
- 若有工具可管理上述待辦清單，請用它追蹤進度。
- 完成每個步驟後，請標示完成並補上摘要。
- 在開始新步驟前，先閱讀目前待辦清單狀態。

溝通規則：
- 避免冗長說明或輸出完整命令結果。
- 若略過步驟，請簡短說明（例如：不需安裝擴充套件）。
- 除非被要求，否則不要解釋專案結構。
- 說明保持精簡且聚焦。

開發規則：
- 除非使用者另有指定，否則使用 '.' 作為工作目錄。
- 除非明確要求，避免加入媒體或外部連結。
- 使用占位符時需註明日後要替換。
- VS Code 擴充套件專案才可使用 VS Code API 工具。
- 專案建立後已在 VS Code 中開啟，不要再建議重新開啟。
- 若專案建立資訊有額外規則，必須嚴格遵循。

資料夾建立規則：
- 一律以目前目錄作為專案根目錄。
- 若執行終端機命令，請使用 '.' 以確保工作目錄正確。
- 除非使用者明確要求，不要建立新資料夾（.vscode 用於 tasks.json 例外）。
- 若 scaffolding 指示資料夾名稱不正確，請告知使用者建立正確資料夾後再重新在 VS Code 開啟。

擴充套件安裝規則：
- 只安裝 get_project_setup_info 指定的擴充套件，不得安裝其他。

專案內容規則：
- 若使用者未提供細節，預設建立 "Hello World" 起始專案。
- 除非明確要求，避免加入任何連結或整合。
- 除非明確要求，避免產生圖片、影片或其他媒體檔案。
- 若必須使用媒體占位符，需註明日後替換。
- 所有產生的元件需符合使用者需求。
- 若功能未確認，不要自行加入，先詢問使用者。
- VS Code 擴充套件專案才可使用 VS Code API 工具查詢相關參考。

任務完成規則：
- 任務完成條件：
  - 專案已成功建立並且編譯無誤
  - 專案內存在 .github/copilot-instructions.md
  - README.md 存在且為最新
  - 已提供清楚的偵錯或啟動指引

開始新任務前，先更新上述計畫進度。
-->
- 依序完成每個清單項目。
- 溝通保持精簡與聚焦。
- 遵循良好開發實務。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/killgavin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
