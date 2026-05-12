# Stage 3 — VPick 專案建立 + Role Board 並行生成

## 目的
建立全新 VPick 專案,並在專案內**並行**生成所有後續會用到的「視覺一致性 reference 圖節點」:
- **1 張角色總圖**(所有角色合併:大頭正面照 + 服裝)
- **N 張環境平面+寫實圖**(每個環境一張,含俯視平面圖 + 寫實場景圖)
- **1 張道具圖**(若有關鍵道具)

## 操作順序

### Step 3.0 — 建立 VPick 專案(此階段第一個動作)

```
vpick:create_project(name="[依故事內容命名,例:雨夜街頭_storyboard]")
vpick:switch_project(project_id=...)
```

告訴使用者:
```
已建立 VPick 專案:[name]
專案 ID:[project_id]

接下來會「並行」生成 Role Board:
- 1 張 角色總圖(所有角色 + 服裝 + 大頭照)
- N 張 環境圖(每個環境一張,含平面圖 + 寫實圖)
- 1 張 道具圖(若有)

所有圖片同時起跑,完成一張就顯示一張。
你可以隨時指定「重生第 N 張」單獨重做,其他保留。
全部圖滿意後請說「全部確認」,我才會進入 Stage 4。
```

### Step 3.1 — 並行建立所有 image generator 節點

**在同一輪內**,一次性建立所有節點並啟動執行:

```
# 角色總圖
# 收集所有「使用者有提供 reference_image_url」的角色 URL,沒提供的不放
vpick:add_node(type="image_generator", model="gpt-image-2", label="ref_characters_all")
vpick:update_node(
  node_id=...,
  prompt=<character_all_prompt>,
  reference_images=[c.reference_image_url for c in characters if c.reference_image_url]  # 可能是空陣列
)
vpick:run_image_generator(node_id=...)

# 環境圖(每個 env 一張)
vpick:add_node(type="image_generator", model="gpt-image-2", label="ref_env_1")
vpick:update_node(node_id=..., prompt=<env_1_prompt>)
vpick:run_image_generator(node_id=...)

vpick:add_node(type="image_generator", model="gpt-image-2", label="ref_env_2")
...

# 道具圖
vpick:add_node(type="image_generator", model="gpt-image-2", label="ref_props")
vpick:update_node(node_id=..., prompt=<props_prompt>)
vpick:run_image_generator(node_id=...)
```

**全部 `run_image_generator` 都先呼叫**,不等任何一張完成。記下每個 node_id。

### Step 3.2 — 即時呈現完成的圖片

每張圖完成就立即顯示給使用者:

```
✓ ref_characters_all 完成
[顯示圖片]

(其他圖仍在生成中...)
```

依完成順序逐張顯示,不用按建立順序。

### Step 3.3 — 全部完成後的總結

所有圖片完成後:

```
Stage 3 全部 Role Board 已生成:

| 圖 | 標籤 | 節點 ID | URL |
|---|---|---|---|
| 角色總圖 | ref_characters_all | node_xxx1 | [URL] |
| 環境 1 | ref_env_1 | node_xxx2 | [URL] |
| 環境 2 | ref_env_2 | node_xxx3 | [URL] |
| 道具圖 | ref_props | node_xxx4 | [URL] |

請逐張確認:
- 滿意 → 說「全部確認」進入 Stage 4
- 要改 → 告訴我「重生 ref_env_1,平面圖再清楚一點」,我會重跑該節點(其他保留)
```

### Step 3.4 — 處理重生請求

使用者要求重生某張時:
1. 取得該節點 node_id
2. 視需要 `update_node` 修改 prompt(若使用者指定了修改方向)
3. `run_image_generator(node_id=...)`(同一節點重跑)
4. 完成後顯示新圖,等待進一步指示

### Step 3.5 — 進入 Stage 4

使用者說「全部確認」後,讀取 `stage4-storyboard-spec.md`,進入 Stage 4。

## Prompt 模板

### 角色總圖 Prompt(所有角色合併在一張)

```
Professional film pre-production character reference sheet — ALL CHARACTERS in single sheet.

Layout:
- Horizontal layout with characters arranged side by side
- For each character, show:
  * Large head-and-shoulders front view (頂部大頭照)
  * Full body front view
  * Full body side view (3/4 angle)
  * Full body back view
  * Costume detail close-ups (collar, sleeves, shoes)
- Clear name label below each character group
- Clean white / light gray background with subtle grid
- Neutral standing pose, arms relaxed
- Flat even lighting for clear visibility

Characters to include:

{for each character in characters:}
  {character.name} ({character.id}):
    Appearance: {character.appearance}
    Age: {character.age}
    Costume (must match exactly): {character.costume}
    {if character.reference_image_url is not null:}
      Reference image attached: see input image #{index} for facial features and body proportions ONLY
      DO NOT copy costume from reference image — use the costume described above
    {else:}
      No reference image — generate face / body from the appearance description above

Visual style: {setup_config.visual_style}

REFERENCE IMAGE NOTES (僅當有任何角色提供 reference_image_url 時加入):
The attached images are references for FACIAL FEATURES and BODY PROPORTIONS ONLY.
Apply the new costumes described above.
Keep faces and body types consistent with each character's attached reference image.

High readability, cinematic concept art quality, production design reference layout.
No facial expression sheets, no action poses, no environment backgrounds, no props.
```

呼叫時把**所有有提供 `reference_image_url` 的角色**的 URL 一起當 input 餵給 `gpt-image-2`,並在 prompt 中明確指明哪張 reference 對應哪個角色。若沒有任何角色提供 reference,`reference_images` 傳空陣列,prompt 走純文字模式。

### 環境圖 Prompt(每個環境一張,含平面圖 + 寫實圖)

```
Professional film pre-production environment reference board — single environment.
SPLIT LAYOUT into two sections:

TOP HALF: photorealistic cinematic scene of the environment
  Location: {env.location}
  Time: {env.time_of_day}
  Weather: {env.weather}
  Lighting: {env.lighting}
  Color tone: {env.color_tone}
  Visual style: {setup_config.visual_style}
  (No characters, no people)

BOTTOM HALF: top-down architectural FLOOR PLAN of the same environment
  Style: clean black-and-white architectural drawing
  Show: walls, doors, windows, key furniture / landmarks, key positions
  Label key elements in 繁體中文
  Scale indicator if possible
  Layout description: {env.floor_plan_note}

Both halves clearly separated with a divider line and labels:
  "Environment View" (top, in English + 繁體中文)
  "Floor Plan 平面圖" (bottom, in English + 繁體中文)

Single environment name label at the top: {env.name}

High readability. Floor plan helps maintain spatial consistency in subsequent shots.
```

### 道具圖 Prompt

```
Professional film pre-production props reference sheet.
Multiple props arranged on clean light background.
Each prop shown isolated with:
- Front view
- Detail close-up
- Size scale indicator (尺寸標示)
- Name label in 繁體中文

Props:
{for each prop:}
  - {prop.name}: {prop.description}

Visual style: {setup_config.visual_style}
No characters, no people, no backgrounds beyond plain neutral surface.
High readability, cinematic concept art quality.
```

## 重要規則

1. **VPick 專案在 Step 3.0 建立**,不是 Stage 5
2. **並行起跑**:Step 3.1 一次性把所有 `run_image_generator` 都呼叫,不等任何一張
3. **完成順序不可預測**,以實際完成順序顯示給使用者
4. **重生單張時只跑該節點**,其他不動
5. **必須等使用者說「全部確認」才進 Stage 4**——即使所有圖都完成
6. 在 context 中維護節點對應表(node_id ↔ label ↔ output URL),Stage 4、Stage 5 都會引用
7. **角色 reference 圖串接**:把所有「使用者有提供 `reference_image_url`」的角色 URL 當 input 餵給 `gpt-image-2`,並在 prompt 中明確對應關係;無人提供時走純文字
