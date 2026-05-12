# VPick-Short-Video-Skill

把故事腳本透過 5 階段流程,在 VPick 內生成完整短劇影片。**Claude Code CLI 用戶**請用這個版本(skill 形式安裝)。

> 用網頁版 Claude.ai?請改用根目錄的 `VPick-Short-Video-Project/`。

---

## 🚀 快速開始(3 步)

### 1. 確認前置需求

- **Claude Code CLI** 已安裝
- **VPick MCP connector** 已連線(必要,所有節點都靠 `mcp__vpick__*` 工具)
  - 若沒裝,先依 VPick 官方文件設定 connector,並確認 `mcp__vpick__create_project` 可呼叫

### 2. 安裝 skill

把整個 `VPick-Short-Video-Skill/` 複製或 symlink 到 `~/.claude/skills/vpick-storyboard/`:

```bash
# 方式 A:複製(推薦給單機使用)
cp -r VPick-Short-Video-Skill ~/.claude/skills/vpick-storyboard

# 方式 B:symlink(方便日後從 repo 拉新版)
ln -s "$(pwd)/VPick-Short-Video-Skill" ~/.claude/skills/vpick-storyboard
```

確認檔案結構:

```
~/.claude/skills/vpick-storyboard/
├── SKILL.md                          # Skill 進入點 + frontmatter
├── README.md                         # 本檔
└── reference/                        # 5 個階段規格 + schema
    ├── 00-INSTRUCTIONS.md
    ├── stage1-setup-config.md
    ├── stage2-script-json-schema.md
    ├── stage3-role-board-spec.md
    ├── stage4-storyboard-spec.md
    ├── stage5-vpick-assembly.md
    └── image-prompt-schema.json
```

重啟 Claude Code 後 skill 會被自動載入。

### 3. 貼一段故事腳本就開始

在 Claude Code 對話中,直接貼上腳本或說出觸發詞:

```
下雨的夜晚,Anna 撐著紅色雨傘走在巷弄裡,
突然 Bryan 從陰影中出現,說:「我等妳很久了。」
她退後半步,握緊了傘柄...
```

或者:

- 「用 VPick 把這個故事做成短劇」
- 「幫我做 storyboard」
- 「Seedance 短劇」
- 「做一支分鏡」

Claude 會自動觸發 skill 進入 Stage 1。15–20 分鐘後拿到完整短劇 + 所有檔案清單。

---

## 🎬 5 階段流程速覽

| Stage | 動作 | 並行? | 確認門檻 |
|---|---|---|---|
| 1 | 確認影片基本設定(風格、長寬比、角色、是否要 reference 圖/語音) | — | 1 個 JSON |
| 2 | 拆解腳本 → JSON + 表格 | — | 1 個 JSON |
| 3 | `create_project` → 角色總圖 + 環境平面圖 + 道具圖 | ✓ | 全部圖確認 |
| 4 | 各 Part 繁中表格 Storyboard | ✓ | 全部圖確認 |
| 5 | Seedance 2 影片 → 合併 → 列檔(13 個檔案表格) | ✓ | 「開始生成」 |

每階段結束必須等使用者「全部確認」才進下一階段。

---

## 👤 客製你的角色與聲音(重要)

本流程 **預設範例使用 Andy 的男主角頭像 + 聲音**,讓你看到流程跑起來的樣子:

| 範例 | URL |
|---|---|
| Andy 男主角頭像 | `https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png` |
| Andy 男聲 | `https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3` |

> ⚠️ **這只是範例,強烈建議換成你自己的!**

### 換成你自己的(超簡單)

在 Stage 1 設定時,直接告訴 Claude:

```
character_b 的 reference_image_url 換成 https://你的圖.com/me.png
character_b 的 voice_reference_url 換成 https://你的mp3.com/me.mp3
```

或者在 Stage 2 確認 JSON 時直接改:

```json
{
  "id": "character_b",
  "reference_image_url": "https://你的圖片網址.com/me.png",
  "voice_reference_url": "https://你的音檔網址.com/me.mp3"
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
| 頭像圖 | png/jpg,解析度 ≥ 512×512,最佳是 4 面向參考(正面 / 側面 / 背面 / 大頭) |
| 語音 mp3 | 10–30 秒乾淨人聲,單一說話者,無背景音樂,中性語氣 |

### 沒有頭像 / 沒有錄音 也沒關係

填 `null`,流程自動降級:

- 沒頭像 → Stage 3 純文字生成角色總圖
- 沒語音 → Stage 5 Seedance 自動配音(音色不可控)

---

## 🧪 測試建議

**第一次測試**:30 秒短劇、2 Parts、1 角色、1 環境、無對白,驗證:

- VPick 專案有在 Stage 3 開頭建立
- 角色總圖是單張合併(不是 N 張獨立圖)
- 環境圖含俯視平面圖
- Stage 4 表格繁體中文可讀
- Stage 5 並行生成正確

**第二次測試**:加對白並用 Andy 範例 / 自己的 voice URL,驗證 Seedance 有正確帶入語音。
