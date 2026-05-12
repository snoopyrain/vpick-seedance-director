---
name: vpick-storyboard
description: VPick AI 短劇分鏡製作助手 — 把故事腳本透過 5 階段流程(設定 → 拆腳本 → 角色/環境 reference → 分鏡表 → Seedance 2 影片)轉成完整短劇,所有節點都建在 VPick 專案內。觸發條件:當使用者說「做分鏡」、「做短劇」、「storyboard」、「把這個故事做成影片」、「用 VPick 做影片」、「Seedance 短劇」、「分鏡表」、「角色 reference 圖」、「環境 reference」、貼上故事腳本要求做成影片時使用此 skill。
---

# VPick Storyboard Skill — 5 階段短劇前製到影片產出

把使用者提供的故事腳本,透過 5 階段並行確認流程,在 VPick 內建立角色 reference、環境 reference、分鏡表,最終生成 Seedance 2 影片並合併。

## 前置需求(每次啟動前先確認)

- **VPick MCP connector 必須已安裝並可用**(`mcp__vpick__*` 系列工具)
- 若 `mcp__vpick__create_project` 不可呼叫 → 告訴使用者去裝 VPick connector,本 skill 不能繼續

## 階段檔案總覽(全部在 `reference/` 子目錄)

| 檔案 | 用途 | 何時讀取 |
|---|---|---|
| `reference/00-INSTRUCTIONS.md` | 流程控制核心 + 並行確認機制 | 啟動 skill 時先讀 |
| `reference/stage1-setup-config.md` | Stage 1 設定確認 | 進入 Stage 1 時 |
| `reference/stage2-script-json-schema.md` | Stage 2 腳本 JSON + 表格 | 進入 Stage 2 時 |
| `reference/stage3-role-board-spec.md` | Stage 3 建專案 + 並行 role board | 進入 Stage 3 時 |
| `reference/stage4-storyboard-spec.md` | Stage 4 並行分鏡表 | 進入 Stage 4 時 |
| `reference/stage5-vpick-assembly.md` | Stage 5 並行影片 + 合併 + 列檔 | 進入 Stage 5 時 |
| `reference/image-prompt-schema.json` | 所有 prompt 模板 + 欄位定義 | Stage 3/4/5 都會引用 |

## 核心原則(每次都遵守)

1. **嚴格按 5 階段順序**,不可跳階段、不可合併階段
2. **每階段結束必須等使用者「全部確認」才能進入下一階段**(確認詞:確認 / OK / 繼續 / 沒問題 / 下一步 / 全部確認 / 都好)
3. **Stage 5 `run_video_generator` 是最終硬門檻**:必須等使用者明確說「開始生成」才呼叫
4. **每次對話 = 全新 VPick 專案**,Stage 3 第一步永遠是 `mcp__vpick__create_project`
5. **角色 / 語音 reference 由使用者於 Stage 1 設定**:內建一組 Andy 男主角範例(URL 見下方),使用者可換成自己的;Stage 1 必須主動告知「Andy 只是範例,建議換成你自己的」。若使用者最終填 `null`,流程自動降級為純文字 / 自動配音
6. **一致性鎖定**:Stage 4 / 5 所有素材必須沿用 Stage 3 確認過的角色、服裝、道具、環境、光線、色調

### 內建 Andy 範例 URL(僅供示範,Stage 1 須提醒使用者換成自己的)

| 用途 | URL |
|---|---|
| 男主角頭像 | `https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png` |
| 男主角聲音 | `https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3` |

## 啟動流程

當使用者貼上故事腳本或要求開始時:

1. 先讀 `reference/00-INSTRUCTIONS.md` 確認最高規則
2. 讀 `reference/stage1-setup-config.md`,依該檔指示提問,**不要在第一輪就產 JSON 或圖片**
3. 後續每個 Stage 切換時,讀對應的 stage{N} md 檔
4. Stage 3/4/5 並行生成時,`mcp__vpick__run_image_generator` / `run_video_generator` 同時起跑,完成一張立即呈現

## 並行 + 連續確認機制(Stage 3/4/5 通用)

- **同時起跑**:該階段內所有圖片 / 影片任務同時呼叫 `run_*`,不等前一張
- **即時呈現**:每張完成立刻顯示給使用者
- **隨時可重生**:使用者可說「重生 ref_env_1」 / 「Part 2 重做」,只跑該節點
- **必須等「全部確認」**:即使全部完成,沒有明確確認不可進下階段

## 不要做的事

- 不要在 Stage 1/2 就呼叫 `mcp__vpick__create_project` 或任何 `run_*`
- 不要在 Stage 3/4 跳去呼叫 `run_video_generator`
- 不要假設使用者一定要用 Andy 範例 — **永遠在 Stage 1 問,並主動建議換成自己的**
- 不要把任何階段內容寫入 memory(這是 ephemeral 流程,結束後不需要保留)
