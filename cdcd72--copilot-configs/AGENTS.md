# PowerShell 規則（Windows + pwsh 7+）

## Shell 偏好設定

- **預設禁止呼叫 Bash Tool。** 需要執行 shell 指令時，只能使用 PowerShell Tool（pwsh.exe）——這是工具層的硬性限制。Unix 常用語法（`ls | head`、`grep`、`curl`、`find`、`cat`）不是藉口，應改用對應的 cmdlet（`Get-ChildItem | Select-Object -First N`、`Select-String`、`Invoke-WebRequest`、`Get-ChildItem -Recurse`、`Get-Content`）。
- 列出檔案用 Glob，讀取檔案用 Read，搜尋內容用 Grep——專用工具的優先權仍然高於 shell。
- 路徑一律使用 Windows 原生格式（`C:\` 或 `$env:USERPROFILE`），避免使用 `/` 路徑；只有在呼叫 git/POSIX 工具時，才可暫時接受 `/c/...`。

## Bash Tool 例外清單（僅在以下情況允許）

Bash Tool 在預設停用之外，**僅**在這幾種情況可以使用，且必須在呼叫時簡短說明原因：

1. **使用者明確指示**：「用 bash」、「執行 sh 腳本」、「在 MINGW 下執行」等明確要求。
2. **必須依賴 POSIX 行為的腳本**：倉儲中已有的 `.sh` 檔案、`Makefile` 目標、`configure` 腳本——這些用 pwsh 轉譯可能失真，直接用 Bash 執行更穩定。
3. **Git hooks / pre-commit 框架**：很多 hook 假設 `/bin/sh`，用 pwsh 包一層反而會出錯。直接讓 git 自己跑 Bash。
4. **跨平台 CI 腳本的本機重現**：當目標是驗證 `.github/workflows/*.yml` 裡的 shell 步驟是否能跑通時，用 Bash 還原原始行為。
5. **MINGW-only 的二進位檔**：`ssh-agent`、`gpg`、`openssl` 等只有在 MSYS 環境下才能正常運作。

不在上述清單中的情況，一律走 PowerShell。模糊的情況先詢問使用者。

## PowerShell 實務注意

- 檔案寫出預設為 UTF-8 無 BOM（pwsh 7 預設行為）；用 `Out-File` / `Set-Content` 時不要手動加 `-Encoding utf8BOM`，除非目標工具明確要求 BOM。
- 呼叫含有空格的原生 exe 時，使用 call 運算子：`& "C:\Program Files\App\app.exe" arg1`。
- 將 `-`、`@`、`--` 開頭的參數傳給原生指令時，使用 stop-parsing token：`git log --% --format=%H`。
- 多行字串（commit message、檔案內容）使用單引號 here-string，**閉合 `'@` 必須頂格**：

  ```powershell
  git commit -m @'
  feat: add foo

  body line
  '@
  ```

- `-ErrorAction SilentlyContinue` 只會壓制輸出，不會改變退出碼；若要真正吞掉錯誤，應使用 `try { ... -ErrorAction Stop } catch {}`。
- 嚴禁 `Invoke-Expression` 拼接使用者輸入——這會有命令注入風險。
- 嚴禁 `New-Item -Force` 用在已存在的檔案上，因為會清空內容。

---
> Source: [cdcd72/copilot-configs](https://github.com/cdcd72/copilot-configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
