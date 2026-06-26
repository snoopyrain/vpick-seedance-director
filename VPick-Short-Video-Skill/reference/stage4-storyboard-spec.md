# Stage 4 — Storyboard 表格式並行生成(繁體中文)

## 目的
**並行**生成所有 Part 的 Storyboard 表格圖。每張為「直式長方形表格」,內含該 Part 全部 shots(4–7 格),每格 = 標頭 + 左側示意圖 + 右側欄位文字。**所有文字使用繁體中文**。

> 🎬 **分鏡表是 Stage 5 影片的構圖藍圖,要用導演工法寫(見 `seedance-director-craft.md`)。** 表格右側欄位不是流水帳:
> - **動作**欄寫 blocking + 表演 beat(eye-line、micro-pause、呼吸),不是「他很緊張」→ 而是「瞳孔擴大,呼吸停頓,握緊傘柄」
> - **運鏡**欄寫有動機的運鏡(「slow push-in as she realizes」),不是「電影感運鏡」
> - **燈光**欄對齊全局 8K 寫實調(contre-jour、自然光、60:30:10),寫出窗在哪、光從哪來
> - 左側示意圖要畫出**空間 blocking**(角色相對位置,依環境平面圖),不是隨意站位
> - 插圖風格 = 寫實電影感(對齊鎖定的 8K photoreal 影片風格)

## 表格式 Storyboard 設計

### 整體版面

- **方向**:直式長方形(portrait,9:16 或更窄)
- **格數**:該 Part 的 shots 數量(4–7 格)
- **每格內部結構**:標頭橫條 + 左圖(16:9)+ 右側欄位文字
- **所有標籤與內文均為繁體中文**

### 每一格內容範例(繁體中文)

```
┌──────────────────────────────────────────────────┐
│ 鏡頭 1（CUT 1）                                    │
├─────────────────────────┬────────────────────────┤
│                         │ 景別:極遠景 EWS         │
│   [示意圖 16:9]          │ 運鏡:靜止鏡頭,讓身影逼近 │
│   (依 shot.visual_       │ 燈光:橘黃路燈,身影成剪影 │
│    content 生成)         │ 動作:character_b 緩步走出,目光鎖定│
│                         │ 聲音:腳步聲、雨聲        │
│                         │ 對白:—                  │
└─────────────────────────┴────────────────────────┘
```

> ⏱️ 格子標頭只標「鏡頭 N(CUT N)」,**不標固定秒數 / timecode**——每個 Part = 15 秒 clip,cut 之間的節奏由 Seedance 判斷(v4 改變)。

文字必須清楚可讀(給 Seedance 看)。
- 標籤(欄位名稱)用較小但加粗的繁體中文
- 內容文字用普通字重、中等大小
- 字體無襯線、繁體中文
- 背景:淺米色或淺灰

## 操作順序(並行)

### Step 4.1 — 並行建立所有 Storyboard 節點

**在同一輪內**,一次性建立所有 Part 的 storyboard 節點並啟動執行:

```
for each part in story_structure:
  # 取得該 Part 對應的 reference URLs
  ref_chars_url = output URL of ref_characters_all
  ref_env_urls = [output URLs of ref_env_X for X in environments_in_part]
  ref_props_url = output URL of ref_props (if props_in_part exists)
  
  # 建立節點
  node = vpick:add_node(type="image_generator", model="gpt-image-2", label=f"storyboard_part_{part.part}")
  vpick:update_node(
    node_id=node,
    prompt=<storyboard_prompt for this part>,
    reference_images=[ref_chars_url] + ref_env_urls + ([ref_props_url] if exists else [])
  )
  vpick:run_image_generator(node_id=node)
```

**全部 `run_image_generator` 都先呼叫**,不等完成。記下每個 node_id。

### Step 4.2 — 即時呈現完成的圖片

每張完成就立即顯示:

```
✓ storyboard_part_2 完成 (0:15-0:30,含對白)
[顯示圖片]

(其他 Part 仍在生成中...)
```

### Step 4.3 — 全部完成後總結

```
Stage 4 全部 Storyboard 表格已生成:

| Part | 時間範圍 | 節點 ID | URL |
|---|---|---|---|
| 1 | 0:00–0:15 | node_yyy1 | [URL] |
| 2 | 0:15–0:30 | node_yyy2 | [URL] |
| 3 | 0:30–0:45 | node_yyy3 | [URL] |
| 4 | 0:45–1:00 | node_yyy4 | [URL] |

請逐張確認:
- 文字是否清楚可讀(繁體中文)
- 分鏡視覺方向是否符合期望
- 角色與環境一致性

滿意 → 說「全部確認」進入 Stage 5
要改 → 告訴我「重生 Part 2,鏡頭 3 的角色表情改成憤怒」,我會重跑該節點
```

### Step 4.4 — 處理重生請求

使用者要求重生某張時:
1. 取得該 storyboard 節點 node_id
2. 視需要修改 prompt(依使用者指示)
3. `run_image_generator(node_id=...)`(同一節點重跑)
4. 完成後顯示新圖

### Step 4.5 — 進入 Stage 5

使用者說「全部確認」後,讀取 `stage5-vpick-assembly.md`,進入 Stage 5。

## Prompt 模板(繁體中文版表格)

```
Professional film pre-production STORYBOARD TABLE in 繁體中文.
VERTICAL portrait orientation, tall rectangle layout.
Single page with {N} cells stacked vertically (one cell per shot, N is between 4 and 7).

Header row at top of page (in 繁體中文):
  "Part {part_number} 分鏡表 | {time_range} | 專案:{project_name}"

Each cell structured as:
  - Cell header bar (top, in 繁體中文): 
    "鏡頭 {n}(CUT {n})"   (NO fixed seconds / timecode — pacing is decided by Seedance within the 15s part)
  - LEFT side of cell body: 16:9 cinematic frame illustration of the scene
  - RIGHT side of cell body: labeled text fields IN 繁體中文

Right-side text fields (each on its own line, label bold, value regular, ALL IN 繁體中文):
  景別:{shot.shot_size_abbr}({對應的繁體中文景別名稱})
  運鏡:{shot.camera_movement translated to 繁體中文}
  燈光:{shot.lighting}
  動作:{shot.character} — {shot.action}
  聲音:{shot.audio}
  對白:{shot.dialogue or "—"}

Cell illustrations (sequenced top to bottom):
  {for each shot in shots:}
    Cell {shot.shot}: {shot.visual_content}
      Character: {shot.character}, referencing the attached character total reference image
      Action: {shot.action}
      Lighting: {shot.lighting}
      Environment: see attached environment reference image (use both photo and floor plan for spatial accuracy)

景別繁體中文對照(在 prompt 內務必使用對照表):
  EWS = 極遠景
  WS = 遠景
  MS = 中景
  CU = 特寫
  ECU = 大特寫
  OTS = 過肩鏡頭
  POV = 主觀視角
  LA = 仰角
  HA = 俯角

運鏡繁體中文對照範例(視 prompt 中實際運鏡翻譯):
  Slow dolly in → 緩慢推軌
  Tracking shot → 跟拍
  Pan right → 右搖
  Tilt up → 上仰
  Handheld → 手持
  Static → 靜止鏡頭
  Push in → 推近
  Pull back → 拉遠

Style requirements:
  - Illustration style: {setup_config.visual_style}
  - Table background: light cream / off-white paper
  - Thin dark gray cell borders
  - Sans-serif typography for 繁體中文
  - Labels in bold for scannability
  - TEXT MUST BE CLEAR AND READABLE in 繁體中文 — this table is reference material for Seedance video AI
  - Illustrations must respect the floor plan from environment reference for spatial accuracy

CONSISTENCY LOCK (use attached reference images):
  - Same character face, costume, hairstyle as the character total reference
  - Same environment lighting, color tone, and spatial layout as environment reference (including floor plan)
  - Same props as reference
  - DO NOT redesign any visual element

DO NOT include: title slides, decorative borders, watermarks, logos, signatures.
DO NOT use English text in cell content — use 繁體中文 throughout.
```

## 重要規則

1. **並行起跑**:Step 4.1 一次性把所有 Part 的 `run_image_generator` 都呼叫
2. **所有文字使用繁體中文**(欄位標籤 + 內容 + 標頭)
3. **沿用 Stage 3 的所有 reference**:角色總圖 + 對應 Part 的環境圖(含平面圖) + 道具圖
4. **環境平面圖是空間一致性的關鍵**——prompt 中明確指示「respect the floor plan for spatial accuracy」
5. **表格內文字必須清楚可讀**——Stage 5 會把這張當 Seedance 構圖 reference
6. **每個 Part 的 shots 數量在 4–7 之間**,prompt 中要動態填入正確的 `N`
7. **必須等使用者說「全部確認」才進 Stage 5**
8. 在 context 中維護 storyboard 節點對應表(node_id ↔ part_number ↔ output URL)
