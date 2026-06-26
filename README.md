# VPick Seedance Director

把故事腳本透過 5 階段流程,在 VPick 內生成完整短劇影片(角色 reference → 環境 reference → 分鏡表 → Seedance 2 影片 → 合併輸出)——**內建電影導演級的 Seedance prompt 工法**。

> 🎬 **這是 VPick 短劇流程 × Higgsfield「Seedance Shotlist Director」導演工法的整合版。** VPick 負責穩定的 5 階段管線(reference 一致性鎖、並行生成、自動合併);導演工法負責把每段影片寫成有 blocking、表演 beat、有動機運鏡的 `STYLE PREFIX → Characters → Scene → CUT` prompt。影片風格鎖定 **8K 電影寫實**。原 HIG skill 的 HTML shotlist 輸出已移除——產物直接是 VPick `video_generator` 節點的 prompt。

> ⚠️ **必要前置**:VPick MCP connector 必須先安裝並連線到 Claude。本 agent 所有圖片 / 影片節點都靠 `mcp__vpick__*` 工具執行,沒有 connector 就跑不動。

---

## 🎯 兩種使用方式,挑一種

| 模式 | 資料夾 | 適合誰 |
|---|---|---|
| **網頁版 Claude.ai** | [`VPick-Short-Video-Project/`](./VPick-Short-Video-Project/) | 在 claude.ai 用 Project Files。上傳 7 個檔到 Project Files,Instructions 貼 `00-INSTRUCTIONS.md` |
| **Claude Code CLI** | [`VPick-Short-Video-Skill/`](./VPick-Short-Video-Skill/) | 用終端機的 Claude Code。複製整個資料夾到 `~/.claude/skills/vpick-seedance-director/` |

兩邊內容**完全等價**,只是封裝方式不同。詳細安裝步驟在各自的 README。

---

## 🚀 怎麼開始(超簡單)

1. 依上面挑選的模式裝好(看對應資料夾的 README)
2. 直接貼一段故事腳本到對話框,例如:

   > 下雨的夜晚,Anna 撐著紅色雨傘走在巷弄裡,
   > 突然 Bryan 從陰影中出現,說:「我等妳很久了。」
   > 她退後半步,握緊了傘柄...

3. Claude 會自動進入 Stage 1 確認設定(風格、長寬比、角色、是否要用 reference 圖/聲音),一路按指示確認下去
4. 15–20 分鐘後拿到完整短劇 + 13 個檔案清單(reference 圖、分鏡表、各段影片、合併成品)

---

## 🎬 5 階段流程速覽

| Stage | 動作 | 並行? | 確認門檻 |
|---|---|---|---|
| 1 | 確認影片基本設定 | — | 1 個 JSON |
| 2 | 拆解腳本 → JSON + 表格 | — | 1 個 JSON |
| 3 | `create_project` → 角色總圖 + 環境平面圖 + 道具圖 | ✓ | 全部圖確認 |
| 4 | 各 Part 繁中表格 Storyboard | ✓ | 全部圖確認 |
| 5 | Seedance 影片(導演級 prompt,含對白)→ 合併 → 列檔 | ✓ | 「開始生成」+ 最終檔案清單 |

每階段結束會等使用者「全部確認」才進下一階段。Stage 5 `run_video_generator` 必須等使用者明確說「開始生成」才呼叫。

---

## 🎬 導演工法(整合 Higgsfield Seedance Shotlist Director)

本 repo 的靈魂是 `seedance-director-craft.md` —— 把 Higgsfield「Seedance Shotlist Director」skill 的電影導演工法,整合進 VPick 的 5 階段管線:

- **影片風格鎖定 8K 電影寫實**(8K IMAX photoreal Style Prefix,逐字套用到每段影片)
- **每段 prompt 用 `STYLE PREFIX → Characters → Scene → CUT 1/2/3` 結構**,寫 blocking、表演 beat(show-don't-tell)、有動機的運鏡、跨 Part 連續性
- **不標固定 per-cut 秒數**:每個 Part = 一段 15 秒 clip,cut 之間的節奏交給 Seedance 判斷,我們只控制戲劇節奏(v4)
- **去除原 HIG 的 HTML shotlist 輸出**:產物直接是 VPick `video_generator` 節點的 prompt 字串

導演工法貫穿 Stage 2(依戲劇節奏拆鏡頭)、Stage 4(分鏡表寫 blocking 與表演)、Stage 5(導演級 video prompt)。

---

## 👤 客製你的角色與聲音

本流程**內建一組 Andy 男主角範例**(頭像 + 聲音),讓你不必準備素材就能跑完整流程看效果:

| 內建範例 | URL |
|---|---|
| Andy 男主角頭像 | `https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png` |
| Andy 男聲 | `https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3` |

> ⚠️ **這只是範例,做你自己的短劇時請換成自己的!**

### 換成自己的有 3 條路

**路線 1:Stage 1 直接告訴 Claude**(最簡單)

```
character_b 的 reference_image_url 換成 https://你的圖.com/me.png
character_b 的 voice_reference_url 換成 https://你的mp3.com/me.mp3
```

**路線 2:Stage 2 確認 JSON 時直接改**

```json
{
  "id": "character_b",
  "reference_image_url": "https://你的圖片網址.com/me.png",
  "voice_reference_url": "https://你的音檔網址.com/me.mp3"
}
```

**路線 3:全部不要參照,純文字生成**

把對應 URL 欄位填 `null`,Stage 3 純文字生成角色,Stage 5 Seedance 自動配音。

### URL 可以放哪?

任何 HTTP/HTTPS 可直接抓的公開連結都行:

- Google Drive 公開分享(改 direct download URL)
- Google Cloud Storage / S3 / Dropbox 公開連結
- 自己的網站、圖床
- 任何 png / jpg / mp3 直連

### 建議規格

| 類型 | 建議 |
|---|---|
| 頭像圖 | png/jpg,≥ 512×512,最佳是 4 面向(正面 / 側面 / 背面 / 大頭) |
| 語音 mp3 | 10–30 秒乾淨人聲,單一說話者,無背景音樂,中性語氣 |

---

## 📜 License

依本 repo 根目錄 LICENSE 為準。

---

## 開發者備註

- 本 repo 內容為 prompts / instructions 文件,無程式碼需要編譯
- `VPick-Short-Video-Project/` 與 `VPick-Short-Video-Skill/reference/` 內 8 個檔(7 階段/規格檔 + `seedance-director-craft.md`)內容必須保持同步;若改 prompt 規格請兩邊一起改
- 改動後本地測試流程:用一支 30 秒短劇(2 Parts、1 角色、無對白)跑一次,驗證 5 階段順序、導演級 prompt 結構與 VPick 節點建立正確
