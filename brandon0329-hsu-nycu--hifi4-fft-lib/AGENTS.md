每個 Python 函式都要有 Google 風格的 docstring
新增的套件必須立即更新至環境設定檔，例如 requirements.txt 或 Dockerfile
必須要有 Dockerfile
產生的每個圖表、資料檔案或記錄檔，必須在檔名或標題中自動包含實驗編號、日期及產生的函式名稱。
請額外用一個檔案幫我記錄執行的每個指令與碰到的問題以及更改程式之步驟，例如:實驗步驟.txt 
第一次使用終端機時需要輸入這些環境變數
"""
export XTENSA_SYSTEM=/home/yu-chen/xtensa/XtDevTools/install/tools/RI-2021.6-linux/XtensaTools/config
export XTENSA_BASE=/home/yu-chen/xtensa/XtDevTools/install
export  XTENSA_CORE=hifi4_ss_spfpu_7
"""
這是目前編譯測試的指令
"""
xt-clang fftifft_32_32_test_other_signal1.c     -o fftifft_test_program_nature_q15     -I/home/yu-chen/下載/HiFi4_VFPU_Library_v4_1_0/hifi4_library/include     -L.     -lAll_fft_NatureDSP_Signal-hifi4_ss_spfpu_7_llvm-Xtensa-release     -lm

"""
"""|
xt-run fftifft_test_program_nature_q15
"""

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brandon0329-hsu-NYCU)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/brandon0329-hsu-NYCU)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
