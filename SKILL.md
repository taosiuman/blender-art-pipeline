---
name: blender-art-pipeline
description: "Blender 美术生产管线：把「做一个道具/角色/场景/载具/建筑」做到可交付——命名与 collection 层级、面数/贴图/TD 预算、LOD 与碰撞、UV 与烘焙、材质灯光与渲染、绑定动画、几何节点与 FX、引擎导出与 QA，全部有硬规格，并配 asset_qa.py 一键审计、style_presets.py 一键落地风格。做美术资产、转游戏资产、配导出、跑资产验收、调风格氛围时使用。"
homepage: https://www.blender.org/
author: taosiuman
version: 1.1.2
metadata:
  octop:
    emoji: "🎨"
    label:
      zh: "Blender 美术管线"
      en: "Blender Art Pipeline"
    summary:
      zh: "从需求到可交付资产：规格表 + 流水线 + 审计脚本，游戏/影视两用。"
      en: "Spec-driven art production: standards, stage pipeline, QA auditor."
---

# Blender 美术生产管线

把「做个东西」变成「可交付资产」：有规格、有阶段、有验收 gate。

**生产基准 = Blender 5.3（5.3.0）｜兼容下限 = 5.2 LTS（5.2.4）**
**Last verified**: 2026-11-18 —— `asset_qa.py` 与 `style_presets.py` 在 **5.3.0 / 5.2.4 双构建**无头实测；`style_presets` 的 12 个 look 在 **EEVEE 与 Cycles 两个引擎**各 12/12 通过（无黑屏 / 无过曝 / 无灰平）；文档里的材质骨架与「先选集合再导出」两段片段也用双构建探针跑过（FBX/GLB 只含 `COL_Export` 内对象，集合外对象未写入）

---

## 什么时候用

✅ 用：
- 「做一个道具/武器/角色/载具/建筑/场景」
- 「转成游戏资产」（要 LOD、碰撞、引擎导出）
- 「按参考图做」「要像这张图」
- 「检查这个资产能不能交付」
- 批量命名/层级整理、面数与贴图预算核算

❌ 不用（去别的技能）：
- 只是连不上 Blender、装插件、排 MCP 故障 → `blender-mcp`
- 只是查某个 API 在 5.3 有没有变 → `search_knowledge` / `knowledge/blender/`
- 纯代码架构问题 → `plugin-code-templates.md`

---

## 三条铁律

1. **先量后建** —— 动手前必须有：目标平台、视距、面数档位、贴图预算、交付格式。缺一项就先问，别边做边猜。
2. **非破坏优先** —— 布尔保留 live、修改器不 apply 到最后、`COL_Backup` 存一份、改结构前 `save_as_mainfile` 另存。
3. **有 gate 才算完** —— 交付前跑 `asset_qa.py`，`FAIL` 不许发。没有数字的「做完了」不算完。

---

## Stage 0–7（详细展开见 `references/pipeline.md`）

| Stage | 产出 | 过关判据 |
|---|---|---|
| **0 立项** | Production Brief（平台/视距/档位/预算/格式） | 6 项全填 |
| **1 参考** | Reference Analysis（剪影/比例/相机/光源/材质/配色） | 能写出 3 个剪影形容词 + 主光方向 |
| **2 块面** | 灰模，只有大形 | 剪影截图能与参考并排对应 |
| **3 高/低模** | 高模细节 + 低模拓扑 | 低模三角数 ≤ 预算；无 n-gon 破面 |
| **4 UV+贴图** | 展开 + 烘焙 + ORM | TD 达标、pack > 75%、法线图蓝底 |
| **5 材质灯光** | Master material + 三点光 | PBR 数值对表、无死黑无过曝 |
| **6 绑定/动画**（按需） | 骨骼 + 权重 + Action | ≤4 影响/顶点、极端姿态不塌 |
| **7 优化导出** | LOD + 碰撞 + FBX/GLB | 引擎侧验证矩阵全过 |

> 静帧/影视资产可跳过 6/7 的 LOD 与碰撞，但**命名/层级/材质预算不能跳**。

---

## 硬规格速查（最常被问的 12 个数）

| 项 | 值 |
|---|---|
| 命名前缀 | `COL_` `SM_` `MAT_` `T_` `AN_` `ARM_` `GN_` `CAM_` `LGT_` |
| Collection | `COL_Root → Reference/Blockout/Geo/Collision/Lights/Cameras/Export`（七层） |
| 道具面数 | Hero 15k–50k｜标准 5k–15k｜背景 500–5k｜小物 100–500（三角） |
| 角色面数 | Hero 50k–100k｜NPC 20k–40k｜群众 5k–10k |
| LOD 三角比 | 100 / 40–60 / 15–30 / 5–10 % |
| 贴图 | Hero 2k–4k｜标准 1k–2k｜背景 512–1k |
| Texel Density | Hero 512–1024 px/m｜标准 256–512｜背景 128–256 |
| UV pack 效率 | > 75%，margin 0.002–0.01（lightmap ≥ 0.02） |
| 材质数 | Hero 道具 3–5｜标准 1–2｜角色 3–8｜模块 1（atlas） |
| 骨骼影响 | **每顶点 ≤ 4** |
| 导出 | FBX：Scale 1.0 / FBX All / −Z 前 Y 上 / Apply Unit ON / Path Copy |
| 全场景 | 1 unit = 1 m；网格 scale 必须 (1,1,1) |

完整表（含载具/建筑/武器/植被/UV 接缝/材质 PBR 数值/色温/采样）→ `references/standards.md`

---

## 路由表

| 用户在问 | 读 |
|---|---|
| 命名/层级/预算/单位/枢轴/验收数字 | `references/standards.md` |
| 流程怎么排、Brief 怎么写、多技能怎么组合 | `references/pipeline.md` |
| 建模、硬表面布尔、模块化 kit、植被、雕刻、重拓扑 | `references/modeling.md` |
| UV 接缝、TD、打包、UDIM、烘焙、ORM、图集 | `references/uv-texture.md` |
| 材质、PBR 数值、master material、lookdev | `references/materials-lookdev.md` |
| 灯光、相机、采样、色彩管理、通道、合成 | `references/lighting-render.md` |
| 骨骼、权重、IK/FK、动画时长、毛发、布料 | `references/rig-anim.md` |
| 几何节点、程序化生成、物理/FX | `references/gn-vfx-sim.md` |
| 优化、LOD、碰撞、FBX/GLB/USD、Unity/UE/Godot | `references/optimize-export.md` |
| 缺陷分级、检查清单、QA verdict | `references/qa.md` |
| 按参考图做、像不像、比对循环 | `references/reference-match.md` |
| 风格/世界观/情绪（赛博朋克、侘寂、liminal…） | `references/style-recipes.md`，或直接用 `style_presets.py` 一键套 12 个 look |
| 怎么通过 MCP 在 Blender 里执行 | `references/mcp-execution.md` |
| 「它能做的你都能做吗」/ 覆盖面对照 | `references/coverage-map.md` |

---

## 一键审计（本技能的可执行部分）

```bash
RT=C:\Users\haowe\.octop\portable\runtime\python.exe
BL53=C:\Users\haowe\.octop\agents\6WBCS2\generated\blender53\blender-5.3.0-windows-x64\blender.exe
BL52=C:\Users\haowe\.octop\agents\6WBCS2\generated\blender52\blender-5.2.4-windows-x64\blender.exe

# 审计（只读，不改场景）
"%BL53%" --background "C:\path\asset.blend" --python C:\Users\haowe\.octop\agents\6WBCS2\generated\tools\asset_qa.py -- --tier hero --json out\qa.json
```

输出：`PASS / WARN / FAIL` + 逐项违规清单（对象名 + 实际值 + 期望值）。

| 参数 | 说明 |
|---|---|
| `--tier hero\|standard\|background\|small\|character\|npc\|crowd\|vehicle\|drivable\|weapon\|module\|any` | 选预算档（`--list-tiers` 看全表） |
| `--json <path>` | 结构化报告落盘（`verdict / stats / findings`） |
| `--export-gate` | 追加导出前置检查：`COL_Export` 必须存在且非空 |
| `--strict` | WARN 也算失败（退出码 1） |
| `--thresholds <json>` | 覆盖/新增档位阈值 |
| `--quiet` / `--list-tiers` | 静默 / 列档位 |

退出码：有 `FAIL` → 1；`--strict` 下 `WARN` → 1；未知档位 → 2。

检查项：命名前缀 / 默认名残留 / collection 层级 / 面数（含修改器后 evaluated）/ scale & rotation 是否应用 / 材质数与空槽 / UV 存在与 pack 效率 / **texel density** / LOD 链完整性 / 骨骼影响数 / 零面积面 / 非流形边 / 贴图命名与分辨率 / 数据贴图色彩空间（`_N` `_ORM` 必须 Non-Color）。

在运行中的 Blender 里要审计 → 见 `references/mcp-execution.md` §3.2。

---

## 一键 look 落地（风格从「形容词」变成「可渲染的场景状态」）

```bash
# 列出 12 个内置 look（每行 id + 中文方向）
"%BL53%" --background --factory-startup --python C:\Users\haowe\.octop\agents\6WBCS2\generated\tools\style_presets.py -- --list

# 套到一个场景上：世界色 + 雾 + 三点光（色温换算）+ 相机 + AgX/对比度/曝光
"%BL53%" --background --factory-startup --python ...\style_presets.py -- --look cyberpunk-rain --apply --save out\base.blend --json out\look.json --brief
```

内置 look：`cyberpunk-rain` `neon-noir` `golden-hour` `nordic-overcast` `liminal-fluorescent` `horror-cold` `cottage-warm` `desert-haze` `moonlit-night` `studio-product` `hd2d-diorama` `post-apoc-dust`

| 参数 | 说明 |
|---|---|
| `--look <id>` `--apply` | 套用到当前场景（幂等，重复套不会堆灯） |
| `--replace-lights` | 先删本工具建过的 `LGT_Style_* / FOG_Style` 再重建 |
| `--scale <f>` | 场景尺度因子（默认 1.0 = 约 6 m 空间；大场景给 2–4） |
| `--exposure-add <ev>` | 在预设曝光上叠加，微调亮度不破坏 look |
| `--brief` | 打印该 look 的美术方向（造型/材质/配色 + 参考片），给 brief 与 AI 出图用 |
| `--json <path>` | 落盘实际应用的参数（可 diff、可回填） |
| `--save <path>` | 另存 blend（不动原文件） |

**它生成什么**：`World`（天空色 + 强度）、`FOG_Style`（体积雾盒，可删）、`LGT_Style_Key/Fill/Rim`（AREA/POINT/SPOT + 色温→RGB）、`CAM_Style`（焦距/光圈）、`view_settings`（AgX + look + 曝光）。全部落在 `COL_Lights_Style` / `COL_Cameras_Style` / `COL_Style_Fog` 下，**不碰用户几何**。

> ⚠️ **雾必须用体积盒，不能挂 World Volume**：EEVEE 下世界体积会把整块背景吃掉（实测 sky 强度 2.4 → 渲染亮度 0.0003，全黑）。本工具已按体积盒实现，详见 `references/lighting-render.md` §雾。

---

## 版本守卫

- 5.3 新能力（Lists / Bundles / For Each Element、`PointCloud` Gaussian Splat）**不要**在 5.2 兼容代码里引用
- 引擎标识：Eevee 在 5.x 的 enum id 实测是 **`BLENDER_EEVEE`**（4.2+ 曾用 `BLENDER_EEVEE_NEXT`，名字跨版本会改）→ 写死前先 dump enum，别按记忆写
- **色彩管理 look 名写错不会报错**：`view_settings.look = "AgX - Medium Contrast"`（不存在）会被静默忽略 → 要么先 dump `scene.view_settings.bl_rna.properties['look'].enum_items`，要么像 `style_presets.py` 那样把 fallback 记进报告
- **雾别挂 World Volume**：EEVEE 下世界体积会让背景变黑（见上节）；用体积盒，两个引擎一致
- 导出器参数名跨版本会改 → 调用前 `bpy.ops.<fn>.get_rna_type().properties.keys()` 核一次
- 交付脚本一律双跑：5.3.0 + 5.2.4，任一报错即不算完成

---

## 来源与许可

本技能的**工程规格来自实测与公开文档**；**美术数值参考**（面数档位、TD、贴图预算、风格配方、参考图闭环）部分吸收了 MIT 许可的 [arjun988/blender-skills](https://github.com/arjun988/blender-skills)（v1.3.0，94 技能 / 292 KB）的公开规格，并重写为可执行形式：

- 原文为**纯提示词包**（0 行可执行代码），本技能新增：Blender 版本守卫、bpy 片段、MCP 执行协议、`asset_qa.py` 可执行审计
- 未复制其 60+ 风格包的原文，压缩为 `references/style-recipes.md` 的配方表
- 其 README 宣称的 Poly Haven HDRI / Hyper3D 生成**不在官方 add-on v1.0.3**，本技能不采用
- MIT 许可允许商用与再分发；如需保留完整署名 → 见 `references/NOTICE.md`

---

## 维护

- 改本技能后：版本号三处对齐（`SKILL.md` frontmatter / `README.md` / changelog）→ `publish_skill.py --check` → 双平台发布
- ⚠️ **体积纪律**：ClawHub 曾因单文件 > 扫描预算被判 `CRITICAL / DO_NOT_INSTALL` → **主文件保持 < 30 KB，重内容一律进 `references/`**（本文件当前约 9 KB）
