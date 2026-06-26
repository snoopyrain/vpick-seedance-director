# Stage 5 — Seedance 2 影片並行生成 + 合併 + 最終檔案清單

## 目的
在已建立的 VPick 專案內(Stage 3 已建)新增 Seedance 2 video generator 節點,自動串接 reference 圖與 voice reference(依該 Part 是否有對白),最終確認後**並行**生成所有 Part 影片,合併輸出,並回傳所有生成檔案的完整清單與連結。

> 🎬 **寫 video prompt 前先讀 `seedance-director-craft.md`。** Stage 5 影片品質的關鍵在 prompt 的導演含量 — 每個 video_part 用 `STYLE PREFIX → Characters → Scene → CUT 1/2/3` 結構寫,不是把 shot 欄位平鋪。

## 最高規則(此階段專屬)

- **VPick 專案不重新建立**(Stage 3 已建)
- **節點建立 + 連線階段不執行影片生成**。`run_video_generator` 必須等使用者說「開始生成」才呼叫
- **圖片不重新生成**,只引用 Stage 3/4 已建立的節點 output URL
- **Voice reference 帶入**:依 Stage 2 `speaking_characters` 自動決定該 video node 要附加哪些 voice reference URL

## 預設參數

### Seedance 2 影片設定

| 參數 | 預設值 |
|---|---|
| model | Seedance 2 |
| resolution | 480p |
| audio | 啟用(有聲) |
| duration_per_part | 15 秒 |
| aspect_ratio | 沿用 Stage 1 `setup_config.aspect_ratio` |

### Voice Reference 來源

所有 voice reference URL 由**使用者於 Stage 1 設定**並寫入 Stage 2 JSON `consistency_lock.characters[*].voice_reference_url`。

**內建範例(僅供示範,使用者可換成自己的)**:

| 範例 | URL |
|---|---|
| Andy 男聲 | `https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3` |

**Voice 串接邏輯**:
- 該 Part `speaking_characters` 中的每個角色 → 找到該角色的 `voice_reference_url`(在 Stage 2 JSON 中)
- 若該角色的 `voice_reference_url` 不為 null → 把該 URL 附加到該 video node 的 voice reference / audio input
- 若該角色的 `voice_reference_url` 為 null → 不附加 voice ref,Seedance 自動生成該角色的語音(音色不可控)
- 若 `speaking_characters` 為空 → 完全不附加 voice reference,Seedance 只生環境音

## 操作流程

### Step 5.0 — 影片預設參數確認

```
進入 Stage 5。在建立影片節點前,先確認 Seedance 2 影片預設:

| 參數 | 預設 |
|---|---|
| 模型 | Seedance 2 |
| 解析度 | 480p |
| 聲音 | 啟用 |
| 每段時長 | 15 秒 |
| 長寬比 | {setup_config.aspect_ratio} |
| 總段數 | {parts_count} 段 + 1 合併 |

對白語音 reference 將自動帶入(僅當使用者於 Stage 1 有提供該角色的 voice URL):
{依 Stage 2 speaking_characters 與 character.voice_reference_url 列出每個 Part 會帶入哪些 voice URL}
- Part 1:無對白
- Part 2:character_b 對白 → {character_b.voice_reference_url 或 "Seedance 自動生成"}
- Part 3:character_a 對白 → {character_a.voice_reference_url 或 "Seedance 自動生成"}
- Part 4:雙方對白 → 各自的 voice URL,任一為 null 則該角色語音由 Seedance 自動生成

確認沿用預設 → 回覆「確認」開始建立節點
要修改 → 告訴我哪個參數要改
```

### Step 5.1 — 列出 Stage 3/4 已建立的節點

確認 project_id 並列出所有可用節點(角色總圖、環境圖、道具圖、storyboard 圖)的 node_id 與 output URL。

### Step 5.2 — 建立 Seedance 2 Video Nodes(每 Part 一個)

```
for each part in story_structure:
  vpick:add_node(
    type="video_generator",
    model="seedance-2",
    label=f"video_part_{part.part}",
    data={
      "resolution": "480p",
      "audio_enabled": true,
      "duration": 15,
      "aspect_ratio": "{setup_config.aspect_ratio}"
    }
  )
```

記下每個 video node 的 node_id。

### Step 5.3 — 自動串接 References + Voice

依 Stage 2 每個 Part 的 `characters_in_part`、`environments_in_part`、`speaking_characters` 串接:

```
for each part:
  video_node = video_part_{part.part}
  
  # 圖片 reference
  connect ref_characters_all → video_node
  for each env_id in environments_in_part:
    connect ref_env_{env_id} → video_node
  if props_in_part:
    connect ref_props → video_node
  connect storyboard_part_{part.part} → video_node
  
  # Voice reference(僅當使用者有提供時)
  voice_refs = []
  for each char_id in speaking_characters:
    char = lookup character by id
    if char.voice_reference_url is not null:
      voice_refs.append(char.voice_reference_url)
  
  # 把 voice_refs 寫入 video node 的 data(可能是空陣列)
  vpick:update_node(node_id=video_node, data={"voice_references": voice_refs})
```

### Step 5.4 — 填入每個 video node 的 prompt(導演級工法)

> **寫 prompt 前必讀 `seedance-director-craft.md`。** 每個 `video_part_X`(15 秒)= 一個導演 prompt,該 Part 的 shots = prompt 裡的 CUT 1 / CUT 2 / CUT 3…
> 不要只是把 shot 欄位平鋪堆疊(那是舊版做法)。要寫成有 blocking、表演 beat、eye-line、呼吸、有動機運鏡的導演 prompt。

每個 video node 的 prompt 嚴格照這個結構(這就是填進 `prompt` 欄位的字串):

```
[STYLE PREFIX — craft 檔的 8K IMAX 電影寫實版,逐字放最前面(除非使用者於 Stage 1 提供自訂 prefix)]

Characters:
[只列 characters_in_part。短而具體的 anchor。把前面 Part 帶來的狀態延續進來(濕髮、傷口、情緒),外觀沿用 Stage 3 角色總圖]

Scene:
[1–2 句,依 {environments_in_part 的 floor_plan_note} 寫 geo-spatial blocking — 每個角色相對於場景與彼此的位置。含 time_of_day / location]

CUT 1 — [{shot.shot_size} / 景別, lens feel, {shot.camera_movement}]:
[{shot.visual_content} + {shot.character} {shot.action} 寫成導演 beat:手勢、eye-line、micro-pause、呼吸;攝影機在做什麼;光({shot.lighting})在做什麼;diegetic sound({shot.audio})]

CUT 2 — [下一個 shot,同密度]:
...

CUT 3 — [該 Part 最後一個 shot]:
[若有對白:... and says in {language}: "{shot.dialogue}"]

DIALOGUE VOICE:
{if speaking_characters 非空:}
  Use the attached voice reference audio file(s) for the speaker's voice tone and accent.
  Speaker {character.name}: use voice reference #N
  ({依 Step 5.3 串接的 voice_refs 順序對應編號;台詞原文已寫在對應 CUT 內})

CONSISTENCY:
Maintain exact character appearance, costume, environment spatial layout, color tone ({environments[0].color_tone}), and lighting as shown in the attached reference images (character total sheet + environment + floor plan + storyboard table). Do not redesign visual elements.
```

**導演紀律(細節見 `seedance-director-craft.md`)**:
- Style Prefix 每段逐字重複一次(Seedance 吃單一 prompt,放進每段才不 drift)
- 表演用 show-don't-tell(「jaw tightens, swallows once」不是「looks sad」)
- 每個運鏡有動機;每段設計到填滿 15 秒,不留死白
- Continuity / anchor 不寫成可見區塊,以具體語言融進 Characters 與 CUT
- Prompt 一律英文;對白原文寫在 CUT 引號內並標 `says in {language}`

### Step 5.5 — 建立合併節點

```
vpick:add_node(type="combine" or "video_concat", label="final_combined_video")
for each video_part:
  vpick:connect_nodes(from=video_part_X, to=final_combined_video)
```

(實際工具名稱依 VPick 提供的合併能力選擇,優先用 video concat / combine 類)

### Step 5.6 — Auto Layout + 截圖

```
vpick:auto_layout(node_ids=[全部新建節點])
vpick:get_screenshot()
```

### Step 5.7 — 最終確認(硬門檻)

```
VPick 專案節點建立完成:

專案 ID:{project_id}

| 節點類型 | 標籤 | 數量 |
|---|---|---|
| 角色總圖 | ref_characters_all | 1 |
| 環境圖(含平面圖) | ref_env_1, ref_env_2, ... | {N} |
| 道具圖 | ref_props | {0 或 1} |
| Storyboard 表格 | storyboard_part_1 ~ storyboard_part_{N} | {parts_count} |
| Seedance 影片 | video_part_1 ~ video_part_{N} | {parts_count} |
| 合併節點 | final_combined_video | 1 |

每個影片節點的 reference 串接(依 Stage 2 speaking_characters 與使用者提供的 voice_reference_url):
- video_part_1: 角色總圖 + env_1 + storyboard_part_1 + (無 voice)
- video_part_2: 角色總圖 + env_1 + storyboard_part_2 + character_b 的 voice URL(若有)
- video_part_3: 角色總圖 + env_2 + storyboard_part_3 + character_a 的 voice URL(若有)
- video_part_4: 角色總圖 + env_2 + storyboard_part_4 + character_a 與 character_b 的 voice URL(各自視提供狀況)

[附 VPick 畫布截圖]

請至 VPick 畫布確認:
1. 節點連線是否正確
2. 每個 video_part 的 prompt 與 voice reference 是否齊全
3. Reference 圖串接是否齊全

確認沒問題後回覆「開始生成」,我會「並行」執行所有 Seedance 2 影片生成,再合併。
若需調整,告訴我哪個 Part 要改。
```

### Step 5.8 — 並行執行 Seedance 影片生成(等使用者明確說「開始生成」才執行)

**並行起跑所有 video_part**:

```
# 同時呼叫所有 video_part 的 run
for each video_part:
  vpick:run_video_generator(node_id=video_part_X.node_id)

# 不等任何一個完成,全部先丟出去
```

每支影片完成就即時告知使用者:
```
✓ video_part_2 完成 [URL]
(其他 Part 仍在生成中...)
```

允許使用者在這階段說「重生 video_part_2」,則重跑該節點。

### Step 5.9 — 全部影片完成後,執行合併

全部 video_part 都完成後:
```
vpick:run_combine(node_id=final_combined_video.node_id)
```

取得 `final_combined_video` 最終影片 URL。

### Step 5.10 — 回傳完整檔案清單(v3 重點新增)

最終必須列出**整個專案所有生成的檔案**,以表格形式呈現:

```
🎬 整個 Storyboard Production 流程完成!

VPick 專案 ID:{project_id}
VPick 專案連結:{project URL,若有}

═══════════════════════════════════════════
📁 全部生成檔案清單
═══════════════════════════════════════════

【Stage 3 — Role Board】
| 編號 | 標籤 | 描述 | 連結 |
|---|---|---|---|
| 1 | ref_characters_all | 角色總圖(所有角色 + 服裝 + 大頭照) | [URL] |
| 2 | ref_env_1 | 環境 1(寫實圖 + 平面圖) | [URL] |
| 3 | ref_env_2 | 環境 2(寫實圖 + 平面圖) | [URL] |
| 4 | ref_props | 道具圖 | [URL] |

【Stage 4 — Storyboard 表格】
| 編號 | 標籤 | Part | 時間 | 連結 |
|---|---|---|---|---|
| 5 | storyboard_part_1 | 1 | 0:00–0:15 | [URL] |
| 6 | storyboard_part_2 | 2 | 0:15–0:30 | [URL] |
| 7 | storyboard_part_3 | 3 | 0:30–0:45 | [URL] |
| 8 | storyboard_part_4 | 4 | 0:45–1:00 | [URL] |

【Stage 5 — Seedance 2 影片】
| 編號 | 標籤 | Part | 時間 | 對白語音 | 連結 |
|---|---|---|---|---|---|
| 9 | video_part_1 | 1 | 0:00–0:15 | — | [URL] |
| 10 | video_part_2 | 2 | 0:15–0:30 | (使用者提供 / auto) | [URL] |
| 11 | video_part_3 | 3 | 0:30–0:45 | (使用者提供 / auto) | [URL] |
| 12 | video_part_4 | 4 | 0:45–1:00 | (使用者提供 / auto) | [URL] |

【最終合併影片】
| 編號 | 標籤 | 時長 | 連結 |
|---|---|---|---|
| ⭐ 13 | final_combined_video | 0:00–1:00 | **[最終影片 URL]** |

═══════════════════════════════════════════

共生成 13 個檔案,VPick 專案可隨時編輯任一節點重跑。

如需後續調整(換解析度、改某段、改對白),告訴我即可。
```

## 重要規則

1. **`run_video_generator` 與合併執行 API 在 Step 5.8 之前絕對不可呼叫**
2. **並行起跑**:Step 5.8 一次性把所有 `run_video_generator` 都呼叫
3. **不要重跑 `gpt-image-2`**,所有圖片在 Stage 3/4 已完成
4. **VPick 專案在 Stage 3 已建立**,Stage 5 不重建
5. **Voice reference 自動串接**:依 Stage 2 `speaking_characters` 與角色 `voice_reference_url`(由使用者於 Stage 1 自備,可為 null)
6. **最終必須列出完整檔案清單**(Step 5.10),所有 Stage 3 圖、Stage 4 圖、Stage 5 影片、合併影片
7. **video prompt 用導演工法寫**(Step 5.4 + `seedance-director-craft.md`):STYLE PREFIX → Characters → Scene → CUT,show-don't-tell,運鏡有動機,填滿 15 秒,prompt 英文
