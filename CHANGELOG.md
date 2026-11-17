# CHANGELOG

## 1.1.1 — 2026-11-18

只改「会不会被误执行」的部分，能力零变化。

**修复 / 收紧**

- `references/optimize-export.md`：`use_selection=True` 原先只出现在导出器参数里、没写选择步骤 → 补 `select_collection()` 显式选 `COL_Export`（**不调 `select_all`**），FBX / glTF 两段都标明依赖该选择状态
- `references/materials-lookdev.md`：`make_master_material()` 改为**新建材质后按类型取现成的 Output / Principled 节点**，不再重建节点树（旧写法会丢掉既有贴图与分组）；`use_nodes` 只在 `node_tree is None` 时赋
- `references/reference-match.md` / `standards.md`：把「场景清空」「交付前必须清空」改成**新建空工程 / 该集合必须为空**的表述，避免被读成对用户场景的破坏性操作
- 新增 `skill-card.md`（技能卡片），供 ClawHub 展示

**实测**：改动只涉及文档与片段，`asset_qa.py` / `style_presets.py` 未动，1.1.0 的四组实测结论继续有效（5.3.0×EEVEE / 5.2.4×EEVEE / 5.3.0×Cycles 各 12/12）。新增断言式探针（`raw/_doc_snippets_probe.py`）双构建各跑一次 → 5.3.0 与 5.2.4 **均 PASS**：材质新建后节点为 `[BSDF_PRINCIPLED, GROUP, OUTPUT_MATERIAL]` 且 BSDF→Output 已连；`select_collection("COL_Export")` 选中恰为 `[SM_Export_A, SM_Export_B]`；FBX（12,812 / 12,828 B）与 GLB 均**不含**集合外对象 `SM_Outside`。

## 1.1.0 — 2026-11-18

风格从「形容词」变「可渲染状态」。同时修掉一个会让画面全黑的实现错误。

**新增**

- `generated/tools/style_presets.py`：一键 look 落地。世界色/强度、体积雾盒、三点光（色温→RGB，AREA/POINT/SPOT/SUN）、相机（焦距/光圈/朝向）、色彩管理（AgX + look + 曝光），全部落在 `COL_Lights_Style` / `COL_Cameras_Style` / `COL_Style_Fog`，**不碰用户几何**
- 12 个内置 look：`cyberpunk-rain` `neon-noir` `golden-hour` `nordic-overcast` `liminal-fluorescent` `horror-cold` `cottage-warm` `desert-haze` `moonlit-night` `studio-product` `hd2d-diorama` `post-apoc-dust`
- CLI：`--list` / `--look` / `--apply` / `--replace-lights` / `--scale` / `--exposure-add` / `--brief` / `--json` / `--save`
- `--brief` 输出该 look 的造型语言 / 材质 / 配色 + 参考片，可直接进 Brief 或喂给 AI 出图
- `references/style-recipes.md` §0：12 look 的参数表（由脚本 dump 生成并核对，非手抄）

**修复**

- **EEVEE 世界体积导致全黑**：原先雾挂在 World 的 Volume Scatter 上，实测 sky `#C9B48E` @2.4 渲染平均亮度 **0.0003**（背景被体积吞掉）。改为实体体积雾盒 `FOG_Style`，两个引擎表现一致
- **色彩管理 look 名静默失败**：`view_settings.look` 写不存在的名字（如 `AgX - Medium Contrast`）不报错、直接不生效 → 现在记 `look_wanted` / `look_missing` 到报告，并在 `SKILL.md` 版本守卫里写明
- `World.use_nodes` / `Material.use_nodes` 无条件赋值会在 6.0 弃用 → 改为 `node_tree is None` 才赋值
- 预设亮度重标定：nordic/liminal/desert/studio/post-apoc 的 `bg_strength` 与曝光按实测下调（雾修好后暴露出 4 个 look 灰平/过曝）

**实测**

- 12 look × **5.3.0 / 5.2.4** × **EEVEE / Cycles** 四组组合全部 12/12 通过
- 判定口径：无黑屏（`mean<0.03`）、无过曝（`mean>0.85`）、无灰平（`p95−p05<0.15`）
- 修前：`blown=[desert-haze, studio-product]`、`flat=[nordic, liminal, desert, studio]`；修后：**三项全空**

## 1.0.0 — 2026-11-18

首个版本。目标：把「按参考做资产 / 转游戏资产 / 交付前验收」做成有规格、有 gate 的管线，覆盖 `arjun988/blender-skills`（94 技能，MIT）能做的事，但形态是可执行工程。

**新增**

- `SKILL.md` 入口：三铁律、Stage 0–7、12 个关键数值、路由表、版本守卫
- `references/standards.md`：命名前缀体系、七层 collection、面数/贴图/TD/材质预算、LOD 比例、单位与朝向、导出验证矩阵
- `references/pipeline.md`：每阶段产出与过关判据、Production Brief 模板、多技能组合
- `references/modeling.md`：通用建模、硬表面布尔-bevel 流程、模块化 kit、载具/建筑/植被、雕刻与重拓扑
- `references/uv-texture.md`：接缝规则、TD 公式与实测、打包与 margin、UDIM、烘焙设置、ORM 打包、图集
- `references/materials-lookdev.md`：PBR 数值表、master material 库、材质数预算、lookdev 顺序
- `references/lighting-render.md`：三点光与色温、采样建议、AgX、pass 清单、合成、无头渲染
- `references/rig-anim.md`：骨骼层级、IK/FK 选择、权重 ≤4、动画帧数表、毛发与布料
- `references/gn-vfx-sim.md`：几何节点生成器套路、性能规则、物理/破碎、缓存策略
- `references/optimize-export.md`：优化审计清单、LOD 生成脚本、碰撞代理、FBX/GLB/USD、Unity/UE/Godot 差异
- `references/qa.md`：检查清单、缺陷分级（P0–P3）、QA verdict 模板
- `references/reference-match.md`：参考图匹配九步闭环、分析模板、视觉验收清单
- `references/style-recipes.md`：风格配方表（明确标注为美术方向提示而非工程规格）
- `references/mcp-execution.md`：MCP 执行协议、批量 payload 写法、超时与后台模式的坑
- `references/NOTICE.md`：MIT 来源与许可全文

**工具**

- `generated/tools/asset_qa.py`：只读审计器。命名 / collection / 面数（evaluated）/ 变换是否应用 / 材质数与空槽 / UV 与 pack 效率 / texel density / LOD 链 / 骨骼影响 / 零面积面 / 非流形 / 贴图命名与色彩空间；`--tier` 档位、`--json` 报告、`--export-gate`、`--strict`、`--thresholds`

**实测**

- Blender 5.3.0（`02f0899d78e3`）与 5.2.4 LTS（`cd5dc428cce1`）双构建无头运行
- 构造违规场景 → 逐项命中（超预算、LOD 断号、影响数 6、scale 未应用、pack 效率 0.33…）
- 构造合规场景 → `VERDICT: PASS  FAIL=0 WARN=0`（双构建一致）

**取舍**

- 不引入其 60+ 风格包原文（64% 为 <300 B 占位），压缩为配方表
- 不采用其宣称但官方 MCP v1.0.3 不具备的工具（Poly Haven HDRI / Hyper3D / Hunyuan3D）
- 不复制其 bpy 片段（覆盖度低于 `knowledge/blender/plugin-code-templates.md`），本技能片段均带版本守卫
