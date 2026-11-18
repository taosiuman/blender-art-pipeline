> [!IMPORTANT]
> **This skill is deprecated (2026-11-18).** It has been merged into [`blender-mcp` v4.0.0](https://github.com/taosiuman/blender-skill) as `references/art-*.md` (15 chapters) plus the `asset_qa.py` / `style_presets.py` tools.
> The ClawHub listing has been withdrawn. **Install `blender-mcp` instead** — one skill covers both MCP control and the art production pipeline.
> 中文：本技能已并入 `blender-mcp` v4.0.0（`references/art-*.md`），ClawHub 已下架；请改装 `blender-mcp`。

# Blender Art Pipeline

**Version: 1.1.3** — 2026-11-18｜Blender 5.3（生产基准）/ 5.2 LTS（兼容下限）

**从需求到可交付资产：规格表 + 阶段流水线 + 可执行审计器 + 风格一键落地。**

面向 Blender 5.3（生产基准）与 5.2 LTS（兼容下限）。不是「提示词灵感包」，是可核对的工程规格：每个数字都能被 `asset_qa.py` 验，每一段 bpy 都能在 5.3.0 / 5.2.4 双构建跑。

---

## 它解决什么

模型谁都能摆出来，能不能交付是另一回事：

- 命名乱、collection 乱 → 别人接手就得重做
- 面数/贴图/TD 没预算 → 进引擎掉帧，返工
- UV 接缝开在脸上、lightmap 没留 margin → 烘焙脏
- LOD 链断号、碰撞用高模、枢轴歪 → 引擎里飘
- 「像这张参考图」→ 没有比对标准，来回改十轮

本技能把这些变成**可判定的数字**，并在交付前用脚本卡一道 gate。

---

## 内容结构

```
blender-art-pipeline/
├── SKILL.md                   # 入口：路由表 / 铁律 / Stage 0–7 / 12 个关键数
├── README.md                  # 本文件
├── CHANGELOG.md
└── references/
    ├── standards.md           # 命名 / 七层 collection / 面数 / 贴图 / TD / 单位朝向 / 导出矩阵
    ├── pipeline.md            # Stage 0–7 展开 + Production Brief 模板 + 多技能组合
    ├── modeling.md            # 通用 / 硬表面 / 道具 / 载具 / 建筑 / 植被 / 角色 / 雕刻 / 重拓扑
    ├── uv-texture.md          # 接缝规则 / TD 计算 / 打包 / UDIM / 烘焙 / ORM / 图集
    ├── materials-lookdev.md   # PBR 数值表 / master material / 材质预算 / lookdev 流程
    ├── lighting-render.md     # 三点光 / 色温 / 采样 / AgX / pass / 合成 / 无头渲染
    ├── rig-anim.md            # 骨骼层级 / IK-FK / 权重 ≤4 / 动画帧数 / 毛发 / 布料
    ├── gn-vfx-sim.md          # 几何节点生成器 / 性能规则 / 物理 / 破碎 / 缓存
    ├── optimize-export.md     # LOD 生成 / 碰撞代理 / FBX-GLB-USD / Unity-UE-Godot
    ├── qa.md                  # 检查清单 / 缺陷分级 / QA verdict 模板
    ├── reference-match.md     # 参考图匹配九步闭环 + 分析模板 + 视觉验收
    ├── style-recipes.md       # 风格配方表（美术方向提示，非工程规格）
    ├── mcp-execution.md       # 在本管线里用 MCP 执行的协议与坑
    ├── coverage-map.md        # 能力覆盖对照（对方 94 技能 → 本技能章节 → 可执行支撑）
    └── NOTICE.md              # 第三方来源与 MIT 许可
```

---

## 快速开始

### 1. 定 Brief（30 秒）

任何资产先答这 6 项，缺一项就先问：

```
平台 / 视距 / 面数档 / 贴图预算 / 交付格式 / 是否需要 LOD+碰撞
```

### 2. 查规格

```text
命名？→ references/standards.md §1
面数？→ references/standards.md §3（按档位）
UV 接缝开哪？→ references/uv-texture.md §1
PBR 数值？→ references/materials-lookdev.md §1
导出参数？→ references/optimize-export.md §1
```

### 3. 交付前审计

```bash
BL53="C:\Users\haowe\.octop\agents\6WBCS2\generated\blender53\blender-5.3.0-windows-x64\blender.exe"
QA="C:\Users\haowe\.octop\agents\6WBCS2\generated\tools\asset_qa.py"

"%BL53%" --background "asset.blend" --python "%QA%" -- --tier hero --export-gate --json qa.json
```

输出示例：

```
asset_qa  asset.blend  |  tier=hero  |  Blender 5.3.0
objects=42 meshes=31 tris=38420 mats=6 images=12
--------------------------------------------------------------------
[FAIL] NAME_PREFIX        Cube.001                     对象命名缺少类型前缀 (actual=Cube.001 expected=SM_/CRV_)
[FAIL] INFLUENCES         SM_Hero_Body                 每顶点骨骼影响超过 4 (actual=6 expected=<= 4)
[WARN] TEXEL_DENSITY      SM_Hero_Body                 texel density 低于档位 (actual=192 px/m expected=>= 512 px/m)
[WARN] LOD_GAP            SM_Hero_Body                 LOD 链不连续 (actual=[1] expected=连续 0..N)
--------------------------------------------------------------------
VERDICT: FAIL   FAIL=2 WARN=2
```

退出码：有 `FAIL` → 1，可直接接 CI。

### 4. 风格一键落地（可选）

把「要赛博朋克那种湿冷霓虹」变成可渲染的场景状态（世界/雾/三点光色温/相机/AgX），不动几何：

```bash
SP="C:\Users\haowe\.octop\agents\6WBCS2\generated\tools\style_presets.py"
"%BL53%" --background --factory-startup --python "%SP%" -- --list
"%BL53%" --background --factory-startup --python "%SP%" -- --look cyberpunk-rain --apply --save base.blend --brief
```

12 个内置 look：`cyberpunk-rain` `neon-noir` `golden-hour` `nordic-overcast` `liminal-fluorescent` `horror-cold` `cottage-warm` `desert-haze` `moonlit-night` `studio-product` `hd2d-diorama` `post-apoc-dust` —— 参数表见 `references/style-recipes.md` §0。

### 5. 在运行中的 Blender 里做

MCP 通道（官方 add-on）可直连场景执行 —— 见 `references/mcp-execution.md`。连接与排障不在本技能，见 `blender-mcp`。

---

## 硬规格样例（节选）

| 项 | 值 |
|---|---|
| 命名 | `SM_` `MAT_` `T_` `COL_` `ARM_` `AN_` `GN_` `CAM_` `LGT_`；贴图后缀 `_BC` `_N` `_ORM` |
| Collection | `Reference / Blockout / Geo / Collision / Lights / Cameras / Export` |
| 道具三角面 | Hero 15k–50k｜标准 5k–15k｜背景 500–5k｜小物 100–500 |
| 角色三角面 | Hero 50k–100k｜NPC 20k–40k｜群众 5k–10k |
| LOD | 100 / 40–60 / 15–30 / 5–10 % |
| TD | Hero 512–1024 px/m｜标准 256–512｜背景 128–256 |
| 骨骼影响 | ≤ 4 / 顶点 |
| 导出 | FBX Scale 1.0、FBX All、−Z 前 Y 上、Apply Unit ON |
| 单位 | 1 unit = 1 m，网格 scale 必须 (1,1,1) |

完整表见 `references/standards.md`。

---

## 版本与兼容

| 项 | 值 |
|---|---|
| 生产基准 | Blender **5.3.0**（2026-11-10 发布） |
| 兼容下限 | Blender **5.2.4 LTS** |
| 实测构建 | 5.3.0 `02f0899d78e3` ／ 5.2.4 `cd5dc428cce1`（均便携版，无头跑过） |
| 5.4 | Alpha，只用 `hasattr` 兜底，不做基线 |

`asset_qa.py` 只用 `bpy` 稳定 API（`bpy.data` / `mesh.calc_loop_triangles` / `uv_layers` / `evaluated_get`），**5.3.0 与 5.2.4 双构建实测通过**，PASS 路径与 FAIL 路径都已验证。

`style_presets.py` 的 12 个 look 在 **5.3.0 / 5.2.4 × EEVEE / Cycles** 四组组合下各 **12/12 通过**（检查项：无黑屏 `mean<0.03`、无过曝 `mean>0.85`、无灰平 `p95−p05<0.15`）。

---

## 设计取舍

- **不抄提示词，只抄规格**：原文 94 个技能 / 292 KB / **0 行可执行代码**，其中 64% 的 references < 300 B。我们把有数字的部分重写成规格表 + 脚本，把氛围描述压成配方表。
- **重内容一律进 `references/`**：单文件超扫描预算曾被平台判 `CRITICAL / DO_NOT_INSTALL`，主文件保持 < 10 KB。
- **不宣称官方 MCP 没有的能力**：Poly Haven HDRI、Hyper3D/Hunyuan3D 属社区版 `ahujasid/blender-mcp`，官方 add-on v1.0.3 无此工具。

---

## 许可

本技能代码与文字：随本仓库同许可分发。
美术数值参考来源（MIT）与完整许可文本见 `references/NOTICE.md`。
