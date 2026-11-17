# Blender 美术生产管线 · blender-art-pipeline

把「做个东西」变成「可交付资产」：有规格表、有阶段、有验收 gate，还带两个能跑的工具。

- **版本**：1.1.1
- **适用**：Blender 5.3（生产基准）/ 5.2 LTS（兼容下限）
- **作者**：taosiuman
- **许可**：文档 MIT；两个脚本随本技能分发

## 这个技能解决什么

美术资产做砸，通常不是建模水平问题，而是**没有规格**：面数没预算、命名靠心情、UV 密度靠猜、导出参数靠记忆、验收靠眼缘。本技能把每一步写成可判定的规格，并在两个关键节点给可执行工具：

| 阶段 | 交付物 |
|---|---|
| 需求进来 | Brief + 资产清单（类型/预算/交付格式） |
| 建模前 | 命名规范 + collection 层级 + 面数/贴图/TD 预算表 |
| 资产完成 | `asset_qa.py` 一键审计（11–13 类检查、12 档阈值、JSON 报告、退出码当 gate） |
| 风格对齐 | `style_presets.py` 一键落地（12 套 look：世界/雾/三点光/相机/色彩管理） |
| 出引擎前 | FBX / glTF / USD 标准参数 + LOD / 碰撞 / 单位口径 |
| 验收 | 视觉验收清单（构图、比例、材质、灯光、氛围） |

## 两个工具

```bash
# 资产审计：只读，不改场景，输出 JSON + 非零退出码表示不过 gate
blender --background asset.blend --python generated/tools/asset_qa.py -- --profile game-prop --json report.json

# 风格落地：把一套 look 落到空场景，对象全落在 *_Style 集合，不动用户几何
blender --background --factory-startup --python generated/tools/style_presets.py -- --list
blender --background --factory-startup --python generated/tools/style_presets.py -- --look cyberpunk-rain --apply --save base.blend --brief
```

12 套 look：`cyberpunk-rain` `neon-noir` `golden-hour` `nordic-overcast` `liminal-fluorescent` `horror-cold` `cottage-warm` `desert-haze` `moonlit-night` `studio-product` `hd2d-diorama` `post-apoc-dust`。

## 内容结构

```
SKILL.md              三铁律 + Stage 0–7 + 数值速查 + 路由表 + 两个一键工具
README.md             安装与使用、实测结论
references/
  standards.md        命名、单位、collection、面数/贴图/TD 预算
  pipeline.md         Stage 0–7 阶段定义与 gate
  modeling.md         拓扑、支持边、布尔、LOD
  uv-texture.md       UV 密度、打包、贴图类型、烘焙
  materials-lookdev.md 主材质 + 实例化、材质预算
  lighting-render.md  三点光、HDRI、雾（**别挂 World Volume**）、渲染设置
  rig-anim.md         骨架、权重、动作、导出前检查
  gn-vfx-sim.md       几何节点、FX、缓存
  optimize-export.md  FBX / glTF / USD 参数、LOD、碰撞
  qa.md               验收清单
  reference-match.md  参考图九步闭环（相机匹配 → 比例锁定）
  style-recipes.md    12 套 look 的可套用参数 + 氛围词 → 工程参数
  mcp-execution.md    用官方 MCP 在会话里执行这些脚本
  coverage-map.md     本技能与官方文档的覆盖对照
  NOTICE.md           来源与许可
```

## 已验证

- `asset_qa.py` / `style_presets.py` 在 **5.3.0 与 5.2.4** 双构建无头实跑通过
- 12 套 look 在 **EEVEE 与 Cycles** 各 **12/12** 通过（判定口径：无黑屏 mean<0.03、无过曝 mean>0.85、无灰平 p95−p05<0.15）
- 全部结论标了适用版本与核验日期（2026-11-18）
