# Blender 美术生产管线 · blender-art-pipeline

## Description:

把「做个东西」变成「可交付资产」的美术生产管线：命名与 collection 层级、面数/贴图/TD 预算、LOD 与碰撞、UV 与烘焙、材质灯光与渲染、绑定动画、几何节点与 FX、引擎导出与 QA，全部写成可判定的硬规格，并附两个可执行工具——`asset_qa.py`（资产一键审计）与 `style_presets.py`（12 套 look 一键落地）。

面向 Blender 5.3（生产基准）与 5.2 LTS（兼容下限）；文档与脚本均标注适用版本与核验日期（2026-11-18）。

## Publisher:

- 作者：taosiuman
- ClawHub：https://clawhub.ai/taosiuman
- 源码仓库：https://github.com/taosiuman/blender-art-pipeline

### License/Terms of Use:

文档 MIT；两个脚本随本技能一并分发。

## Use Case:

美术、技术美术、独立开发者用它把「做一个道具/角色/载具/建筑/场景」拆成有 gate 的阶段流程：接需求写 Brief 与预算 → 按命名与层级建模 → UV/烘焙/材质预算 → 灯光与渲染设置 → 绑定动画 → 导出前跑 `asset_qa.py` 拿 JSON 报告与退出码 → 需要定调时用 `style_presets.py` 落到空场景做 look 开发。

## Known Risks and Mitigations:

风险：技能里的导出与清理类指引如果直接套在重要资产上，可能改掉用户的选择状态或工程设置。

缓解：`asset_qa.py` 是**只读审计器**（不改场景）；`style_presets.py` 只往 `COL_Lights_Style` / `COL_Cameras_Style` / `COL_Style_Fog` 里新增对象，**不动用户几何**，且可重复套用。导出片段只改 `select_set` 选择状态，不删对象；在正式资产上执行前先另存或版本化。

风险：导出与 `bpy` 片段对模式、选择、目标引擎、资产结构敏感，照抄参数可能不匹配当前工程。

缓解：先在目标工程的副本上跑一遍，确认导出内容（`use_selection` 只导选中对象）与引擎内的单位/轴向/切线结果，再用于交付。

风险：面数/贴图预算是经验区间，跨品类差异大。

缓解：阈值集中在 `asset_qa.py` 的 12 档 profile 里，可按项目改；预算冲突时以引擎实测性能为准。

## Reference(s):

- [ClawHub skill page](https://clawhub.ai/taosiuman/skills/blender-art-pipeline)
- [Source repository](https://github.com/taosiuman/blender-art-pipeline)
- Blender 官方 API 文档：https://docs.blender.org/api/5.3/

## Skill Output:

**Output Type(s):** [Guidance, Markdown, Code, Configuration]

**Output Format:** [Markdown 规格表与检查清单 + 可直接运行的 Python 片段与两个 CLI 脚本]

**Output Parameters:** [1D]

**Other Properties Related to Output:** [`asset_qa.py` 只读；`style_presets.py` 只在专用集合内新增灯光/相机/雾对象，二者都不会修改用户的 mesh 数据]

## Skill Version(s):

1.1.2 (source: server release evidence)

## Ethical Considerations:

使用者需自行判断这些规格与参数是否适合其项目与客户要求；在正式资产上执行任何改动前，先保留可回退的版本。技能的数值预算来自公开制作实践与实测，不构成对特定引擎性能的保证。
