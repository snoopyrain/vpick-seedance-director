# Stage 2 — 腳本文字版確認

## 目的
將使用者的故事腳本拆解為結構化的多 Part 分鏡 JSON,以「JSON + 易讀表格」雙形式呈現。標註每個 Part 用到哪些角色與環境(供 Stage 5 自動串接 reference 使用)。

> 🎬 **拆鏡頭前先讀 `seedance-director-craft.md` 的「Stage 2 拆鏡頭時怎麼用」段落。** 用導演視角讀腳本:依**戲劇節奏**切 shots(一句重台詞值一個獨立 shot;對峙前的繞圈值一個 shot),不是依秒數平均切。`visual_content` / `action` / `lighting` / `camera_movement` 先帶上導演含量寫,Stage 5 才能無痛展開成 CUT。

## 角色外型 / 語音參照(由使用者於 Stage 1 設定,可選)

每個角色的 `reference_image_url` / `voice_reference_url` 由使用者於 Stage 1 設定。**內建一組 Andy 男主角範例**(僅供示範):

| 內建範例 | URL | 用途 |
|---|---|---|
| Andy 男主角頭像 | `https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png` | 4 面向臉部與體型基準 |
| Andy 男聲 | `https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3` | Stage 5 男主角對白語音 |

> ⚠️ **這只是範例**。使用者可以(也建議)把 `reference_image_url` / `voice_reference_url` 換成自己的頭像 / mp3 URL,做出真正屬於自己的短劇。任何可公開存取的圖片 / 音檔 URL 都可以。

**若有提供 URL(自己的或內建範例)**:
- 該圖只作為臉部 / 體型基準,Stage 3 生成角色總圖時:
- 必須依劇本指定的服裝、髮型、姿態、光線重新生成
- 不可直接複製貼上參照圖
- prompt 中明確說明「reference appearance from attached image, but apply new costume / hairstyle / lighting as described below」

**若未提供 URL** (`reference_image_url: null`):
- Stage 3 純文字生成角色總圖
- 後續所有 shot 以 Stage 3 角色總圖作為鎖定基準

## Shots 拆解規則(v4 修訂 — 不再標固定 per-cut 秒數)

| 項目 | 規則 |
|---|---|
| 每個 Part 時長 | 固定 15 秒(Seedance clip 長度,唯一固定的時間單位) |
| Shots / CUTs 數量 | **4–7 個**(彈性) |
| 每個 shot 的秒數 | **不指定**——只排順序與戲劇 beat,**讓 Seedance 在這 15 秒內自己判斷每個 cut 的長短** |
| 節奏控制 | 靠 CUT 描述的密度暗示(長 hold 描述靜默/呼吸;快 cut 描述連續動作),不是寫死秒數 |

> ⏱️ **v4 重大改變**:舊版(v3)會給每個 shot 標 `duration_seconds`(≥2s)並要求加總 ≤15s。**新版不標 per-cut 秒數**——你控制戲劇節奏,系統控制碼錶。詳見 `seedance-director-craft.md`。

## 操作流程

### Step 2.1 — 拆解腳本

依 Stage 1 確認的 `parts_count` 拆成對應數量 Part。每 Part 內依故事節奏拆 4–7 個 shots(CUTs)。

**硬規則**:
- 每個 Part 固定對應一段 15 秒 Seedance clip
- **不給每個 shot 標固定秒數**——只排順序與戲劇 beat,Seedance 自己判斷 cut 長短
- 所有角色使用固定 ID(`character_a`, `character_b`...),不可混用「男人」「主角」
- 視覺描述必須具體可拍攝,禁止用空泛詞「美麗的」「電影感」

**導演紀律(依戲劇節奏切,不是依秒數平均切 — 見 `seedance-director-craft.md`)**:
- **找戲劇形狀**:這段戲在哪裡轉?落在哪?在哪呼吸?依此決定 shot 的長短與數量,不是把 15 秒平均切成 N 等份。
- **重台詞 / reveal 給獨立 shot**:一句重台詞值一個獨立 shot;一個 reveal 落在單一持續鏡頭,別用多餘 cut 沖淡。
- **動作壓縮、情緒留白**:動作戲用短 shot;告白 / 對峙留長 hold 與靜默 beat。
- **continuity anchor**:每個 shot 的 `character` 欄寫出角色「狀態」(濕髮、流血、力竭、情緒),供 Stage 5 跨 Part 延續。
- **camera_movement 要有動機**:不是「電影感運鏡」,而是「slow push-in as she realizes」這種有理由的運鏡。
- `visual_content` / `action` / `lighting` 先帶上導演含量寫(show-don't-tell),Stage 5 才能無痛展開成 CUT。

### Step 2.2 — 輸出「JSON + 易讀表格」雙形式

先輸出總覽表格(易讀),再附完整 JSON。

#### 表格 A:角色一致性

| ID | 名稱 | 性別 | 角色 | 外觀 | 服裝 | 外型參照 URL | 語音參照 URL |
|---|---|---|---|---|---|---|---|
| character_a | Anna | 女 | 女主角 | 黑髮、25 歲、165cm | 白針織毛衣、淺藍牛仔褲 | (使用者提供 / null) | (使用者提供 / null) |
| character_b | Bryan | 男 | 男主角 | 棕髮、28 歲、180cm | 黑風衣、深色長褲 | Andy 範例 / 使用者自備 | Andy 範例 / 使用者自備 |

#### 表格 B:環境與道具

| ID | 名稱 | 地點 | 時間 | 天氣 | 燈光 | 色調 |
|---|---|---|---|---|---|---|
| env_1 | 雨夜街道 | 都市巷弄 | 23:00 | 中雨 | 橘黃路燈、地面積水反光 | 藍紫冷調 + 橘黃光暈 |
| env_2 | 公寓臥室 | 室內 | 凌晨 1:00 | — | 床頭暖黃燈、藍色月光 | 暖黃 + 冷藍對比 |

道具:
| ID | 名稱 | 描述 |
|---|---|---|
| prop_1 | 紅色折疊雨傘 | 鮮紅尼龍布、木質握把、約 60cm |

#### 表格 C:Part 總覽(關鍵欄位,給 Stage 5 自動串接 reference 用)

| Part | 時間 | 故事目標 | 角色 | 環境 | 道具 | shots 數 | 是否有對白 |
|---|---|---|---|---|---|---|---|
| 1 | 0:00–0:15 | 建立場景、引入女主角、製造懸念 | character_a | env_1 | prop_1 | 5 | 無 |
| 2 | 0:15–0:30 | 衝突發生:男主角出現,首句對話 | character_a, character_b | env_1 | prop_1 | 6 | 男 |
| 3 | 0:30–0:45 | 場景轉換到臥室、回憶閃回 | character_a | env_2 | — | 4 | 女 |
| 4 | 0:45–1:00 | 結局:雙方對話 | character_a, character_b | env_2 | prop_1 | 7 | 男+女 |

#### 表格 D:Part X — Shots 細節(每個 Part 一張表)

範例(Part 2,一段 15 秒 clip,有男主角對白)。**CUT 欄只排順序,不標秒數**——節奏由 Seedance 在這 15 秒內自己判斷:

| CUT | 視覺內容 | 角色與動作(導演 beat) | 景別 | 運鏡(有動機) | 燈光 | 聲音/對白 |
|---|---|---|---|---|---|---|
| 1 | 巷子盡頭出現黑色身影 | character_b 從陰影緩步走出,步伐沉穩 | EWS | Static,讓身影逼近 | 路燈逆光,身影成剪影 | 腳步聲、雨聲 |
| 2 | 雨傘下兩人正面遭遇 | character_a 後退半步,握緊傘柄;character_b 直視不動 | MS | Slow push in,壓迫感 | 路燈側光,兩人臉部各半陰影 | 雨聲變大、心跳低頻 |
| 3 | character_b 嘴部與眼神特寫 | character_b 嘴唇微張,目光鎖定,說出口 | ECU | Static,讓台詞落地 | 重點打光在臉部下半 | 對白:「我等妳很久了」(character_b voice) |
| 4 | character_a 反應特寫 | character_a 瞳孔擴大,呼吸停頓,喉結滾動一次 | CU | Slight handheld,呼吸感 | 路燈頂光,陰影壓眼窩 | 心跳音、雨聲 |
| 5 | 鏡頭拉遠到兩人對峙全身 | character_a 握緊傘把;character_b 紋風不動,僵持 | WS | Slow pull back,拉開張力 | 路燈背光,兩人輪廓清晰 | 雨聲、低頻緊張音樂 |

(5 個 CUT 構成這段 15 秒 clip;每個 cut 多長由 Seedance 依描述的節奏判斷,我們不寫死秒數)

#### 完整 JSON

```json
{
  "stage": 2,
  "project_meta": {
    "type": "short_drama_storyboard",
    "total_duration_seconds": 60,
    "parts_count": 4
  },
  "consistency_lock": {
    "characters": [
      {
        "id": "character_a",
        "name": "Anna",
        "gender": "female",
        "role": "女主角",
        "age": "25",
        "appearance": "黑色長髮、輪廓分明、身高 165cm",
        "costume": "白色針織毛衣、淺藍牛仔褲、白色帆布鞋",
        "personality": "內斂、堅定",
        "reference_image_url": null,
        "voice_reference_url": null,
        "reference_image_note": "外型 / 臉部 / 體型基準,Stage 3 須依劇本換裝。若 null 則純文字生成。"
      },
      {
        "id": "character_b",
        "name": "Bryan",
        "gender": "male",
        "role": "男主角",
        "age": "28",
        "appearance": "棕色短髮、輪廓硬朗、身高 180cm",
        "costume": "黑色風衣、深色長褲、皮鞋",
        "personality": "冷靜、神秘",
        "reference_image_url": "https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png",
        "voice_reference_url": "https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3",
        "reference_image_note": "預填 Andy 範例。使用者可換成自己的頭像 / mp3 URL,或填 null 走純文字 / 自動配音。"
      }
    ],
    "environments": [
      {
        "id": "env_1",
        "name": "雨夜街道",
        "location": "都市巷弄",
        "time_of_day": "深夜 23:00",
        "weather": "中雨",
        "lighting": "橘黃路燈、霓虹反光、地面積水反射",
        "color_tone": "藍紫冷調 + 橘黃光暈",
        "floor_plan_note": "T 字形巷弄,主巷東西向約 20m,北側支巷 8m,巷口有店家招牌"
      },
      {
        "id": "env_2",
        "name": "公寓臥室",
        "location": "室內 25 坪小公寓",
        "time_of_day": "凌晨 1:00",
        "weather": "—",
        "lighting": "床頭暖黃燈、窗外藍色月光",
        "color_tone": "暖黃 + 冷藍對比",
        "floor_plan_note": "方形房間 4x5m,床靠西牆,窗在北牆,門在東牆,書桌在南牆下"
      }
    ],
    "props": [
      {
        "id": "prop_1",
        "name": "紅色折疊雨傘",
        "description": "鮮紅尼龍布、木質握把、約 60cm 長"
      }
    ],
    "consistency_notes": [
      "Same face across all parts — match character reference",
      "Same costume unless story explicitly changes",
      "Same props throughout",
      "Same lighting style and color tone within same environment",
      "Floor plan layout must remain consistent across shots in same environment"
    ]
  },
  "story_structure": [
    {
      "part": 2,
      "time_range": "0:15-0:30",
      "story_goal": "衝突發生:男主角出現",
      "emotional_arc": "警覺 → 對峙",
      "characters_in_part": ["character_a", "character_b"],
      "environments_in_part": ["env_1"],
      "props_in_part": ["prop_1"],
      "has_dialogue": true,
      "speaking_characters": ["character_b"],
      "shots": [
        {
          "shot": 1,
          "visual_content": "巷子盡頭出現黑色身影",
          "character": "character_b 從陰影緩步走出,步伐沉穩",
          "action": "緩步前進,目光鎖定前方",
          "shot_size": "Extreme wide shot",
          "shot_size_abbr": "EWS",
          "camera_movement": "Static,讓身影逼近",
          "lighting": "路燈逆光,身影成剪影",
          "audio": "腳步聲、雨聲",
          "dialogue": null,
          "dialogue_speaker": null
        },
        {
          "shot": 3,
          "visual_content": "character_b 嘴部與眼神特寫",
          "character": "character_b 開口說話",
          "action": "嘴唇微張,目光直視,說出口",
          "shot_size": "Extreme close-up",
          "shot_size_abbr": "ECU",
          "camera_movement": "Static,讓台詞落地",
          "lighting": "重點打光在臉部下半",
          "audio": "雨聲",
          "dialogue": "我等妳很久了",
          "dialogue_speaker": "character_b"
        }
      ],
      "_note": "v4: shot 只排順序(shot: N,= prompt 裡的 CUT N),不帶 time / duration_seconds。每個 Part = 15 秒 clip,shot/cut 之間的長短由 Seedance 判斷。"
    }
  ]
}
```

### Step 2.3 — 確認訊息

```
Stage 2 腳本分鏡完成。請確認:
1. 角色設定(性別、外型、預設參照、語音參照)
2. 環境設定(含平面圖描述 floor_plan_note)
3. Part 總覽:角色組合、環境、shots 數、是否有對白
4. 每個 Part 的 shots(CUTs)細節與順序(不標固定秒數,Part = 15 秒,節奏由 Seedance 判斷)

確認後將進入 Stage 3:
- 第一步建立 VPick 專案
- 接著「並行」生成 角色總圖 + 環境平面圖 + 道具圖
- 每張完成就立即顯示,使用者隨時可指定重生某張
```

## 重要欄位定義

### Shot 欄位

| 欄位 | 必須包含 | 好的範例 | 不好的範例 |
|---|---|---|---|
| `visual_content` | 具體可視覺化的畫面 | 「破敗石廟,黎明,濃霧包圍,地面碎石」 | 「美麗的地方」 |
| `character` | 角色 ID + 狀態 | 「character_a,受傷但冷靜」 | 「一個男人」 |
| `action` | 動作順序與變化 | 「停下腳步,環顧四周,慢慢拔出武器」 | 「他很緊張」 |
| `shot_size_abbr` | 標準縮寫 | EWS / WS / MS / CU / ECU / OTS / POV / LA / HA | 「漂亮鏡頭」 |
| `camera_movement` | 具體運鏡 | 「Slow dolly in toward character_a」 | 「電影感運鏡」 |
| `lighting` | 光源 + 方向 + 顏色 + 強度 | 「冷藍背光穿過濃霧,右側微弱橘色火光」 | 「dramatic lighting」 |
| `audio` | 環境音 + 音效 + 音樂 + 對白 | 「雨聲、心跳音」 | 「緊張音樂」 |
| `dialogue` | 對白原文(若無則 null) | 「我等妳很久了」 | — |
| `dialogue_speaker` | 對白角色 ID | `character_b` | — |

### 環境欄位(v3 新增 floor_plan_note)

| 欄位 | 說明 |
|---|---|
| `floor_plan_note` | 該場景的平面圖文字描述,Stage 3 環境圖會生成俯視平面圖 + 寫實照片 |

### 關鍵欄位:characters_in_part / environments_in_part / speaking_characters

這些欄位是 Stage 5 自動串接 reference 與 voice 的依據:
- `characters_in_part`: 該 Part 出現的所有角色 ID
- `environments_in_part`: 該 Part 使用的所有環境 ID
- `speaking_characters`: 該 Part 有對白的角色 ID 陣列(決定要帶入哪個 voice reference)

範例:
- Part 2 `speaking_characters: ["character_b"]` → 若 `character_b.voice_reference_url` 不為 null,Stage 5 video_part_2 帶入該 URL;否則跳過
- Part 4 `speaking_characters: ["character_a", "character_b"]` → 帶入兩個角色各自的 voice reference URL(任一為 null 則只帶入有提供的那個)

### Step 2.4 — 等待確認
使用者明確確認後,讀取 `stage3-role-board-spec.md`,進入 Stage 3。
