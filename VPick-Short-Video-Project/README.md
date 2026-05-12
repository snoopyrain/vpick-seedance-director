# VPick-Short-Video-Project

把故事腳本透過 5 階段流程,在 VPick 內生成完整短劇影片(角色 reference → 環境 reference → 分鏡表 → Seedance 2 影片 → 合併)。

這個資料夾是 **Claude.ai Project 模式**的全部素材。把它整個丟進一個 Claude Project 即可開用。

> 想用 Claude Code CLI?用根目錄的 `VPick-Short-Video-Skill/` 那一份。

---

## 🚀 快速開始(3 步)

1. **安裝 VPick MCP connector**(必要,本流程所有節點都靠 `mcp__vpick__*`)
2. **建立 Claude Project**:
   - 把 `00-INSTRUCTIONS.md` 內容貼進 Project Instructions 欄位
   - 把另外 6 個檔案上傳到 Project Files:
     - `stage1-setup-config.md`
     - `stage2-script-json-schema.md`
     - `stage3-role-board-spec.md`
     - `stage4-storyboard-spec.md`
     - `stage5-vpick-assembly.md`
     - `image-prompt-schema.json`
3. **貼一段故事腳本進對話框就開始**,例如:

   > 下雨的夜晚,Anna 撐著紅色雨傘走在巷弄裡,突然 Bryan 從陰影中出現,說:「我等妳很久了。」她退後半步,握緊了傘柄...

   Claude 會自動進入 Stage 1 確認設定,15–20 分鐘後拿到完整短劇。

---

## 📁 檔案清單

| 檔案 | 配置位置 | 說明 |
|---|---|---|
| `00-INSTRUCTIONS.md` | Project Instructions | 流程控制核心 + 並行確認機制 |
| `stage1-setup-config.md` | Project Files | Stage 1 開始前設定 |
| `stage2-script-json-schema.md` | Project Files | Stage 2 腳本 JSON + 表格(含 voice ref 串接邏輯) |
| `stage3-role-board-spec.md` | Project Files | Stage 3 建專案 + 並行生 角色總圖/環境平面圖/道具圖 |
| `stage4-storyboard-spec.md` | Project Files | Stage 4 並行生繁中表格式 Storyboard |
| `stage5-vpick-assembly.md` | Project Files | Stage 5 並行影片 + voice ref + 合併 + 最終清單 |
| `image-prompt-schema.json` | Project Files | 所有 prompt 模板 + 欄位定義 |

---

## 🎬 5 階段流程速覽

| Stage | 動作 | 並行? | 確認門檻 |
|---|---|---|---|
| 1 | 確認影片基本設定 | — | 1 個 JSON |
| 2 | 拆解腳本 → JSON + 表格 | — | 1 個 JSON |
| 3 | `create_project` → 角色總圖 + 環境平面圖 + 道具圖 | ✓ | 全部圖確認 |
| 4 | 各 Part 繁中表格 Storyboard | ✓ | 全部圖確認 |
| 5 | Seedance 影片(含對白)→ 合併 → 列檔 | ✓ | 「開始生成」+ 最終檔案清單 |

每階段結束必須等使用者「全部確認」才進下一階段。

---

## 👤 客製你的角色與聲音(重要)

本流程 **預設範例使用 Andy 的男主角頭像 + 聲音**,目的是讓你看到流程跑起來的樣子:

| 範例 | URL |
|---|---|
| Andy 男主角頭像 | `https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png` |
| Andy 男聲 | `https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3` |

> ⚠️ **這只是範例,強烈建議換成你自己的!**

### 換成你自己的(超簡單)

在 Stage 1 確認設定時,把對應角色的兩個 URL 換掉:

```json
{
  "id": "character_b",
  "name": "你的名字",
  "gender": "male",
  "reference_image_url": "https://你的圖片網址.com/me.png",      ← 換這裡
  "voice_reference_url": "https://你的音檔網址.com/me.mp3"       ← 換這裡
}
```

接受的 URL 來源(任一可公開存取的就行):

- Google Drive 公開分享連結(改成 direct download URL)
- Google Cloud Storage / S3 / Dropbox 公開連結
- 你自己的網站或圖床
- 任何 HTTP/HTTPS 可直接抓的 png / jpg / mp3

### 建議規格

| 類型 | 建議 |
|---|---|
| 頭像圖 | png/jpg,解析度 ≥ 512×512,最佳是 4 面向參考(正面 / 側面 / 背面 / 大頭)。臉部清晰、光線均勻 |
| 語音 mp3 | 10–30 秒乾淨人聲,單一說話者,無背景音樂,中性語氣 |

### 沒有頭像 / 沒有錄音 也沒關係

把對應欄位填 `null`,流程會自動降級:

- 沒頭像 → Stage 3 純文字生成角色總圖,後續所有 shot 鎖這張
- 沒語音 → Stage 5 Seedance 自動配音(音色不可控)

---

## 🧪 測試建議

**第一次測試**:30 秒短劇,2 Parts、1 角色、1 環境、無對白,驗證:

- VPick 專案在 Stage 3 開頭有建立
- 角色總圖是單張合併(不是 N 張)
- 環境圖含俯視平面圖
- Stage 4 表格繁體中文可讀
- Stage 5 並行生成正確跑

**第二次測試**:加入 Andy 範例或自己的 voice URL,驗證 Seedance 有正確帶入語音。

---

## v3 主要變動(對比 v2)

| 項目 | v2 | v3 |
|---|---|---|
| Stage 3 角色圖 | 每角色一張 | **全部角色合併一張總圖**(含大頭照 + 服裝) |
| Stage 3 環境圖 | 寫實圖 | **寫實圖 + 俯視平面圖**(避免空間錯亂) |
| Stage 3 生成順序 | 逐張等確認 | **並行起跑,完成即顯示,等「全部確認」** |
| Stage 4 文字 | 英文標籤 | **繁體中文標籤與內容** |
| Stage 4 生成順序 | 逐 Part 等確認 | **並行,等「全部確認」** |
| Shots 數量 | 固定加總 = 15 秒 | **4–7 個,最短 2 秒,加總 ≤ 15 秒** |
| Stage 5 影片生成 | 依序生成 | **並行,完成即顯示** |
| 對白 / 語音 | 未處理 | **可帶入使用者自備的 voice reference** |
| 最終輸出 | 影片連結 | **完整檔案清單(13 個檔案以表格列出)** |
