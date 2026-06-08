# Design Draft Codex Skill

这是一个用于 Codex 的设计稿还原 Skill，主要用于通过 Chrome DevTools MCP 读取 Mockplus、CoDesign 以及类似的浏览器设计稿页面，并将设计稿按较高视觉还原度恢复到前端项目文件中。

本仓库面向两个场景：

1. 同事在 GitHub 上查阅 Skill 的用途、安装方式和使用说明
2. Codex 直接从 `skills/design-draft` 目录安装这个 Skill

## 能做什么

`design-draft` 会引导 Codex 按设计稿还原流程工作，而不是只看截图粗略仿写页面。它重点处理：

- 根据用户描述定位目标设计稿标签页
- 通过 Chrome DevTools MCP 检查 Mockplus 或 CoDesign 页面
- 在写代码前确认要还原的业务内容区域
- 按区域和模块逐步还原，而不是一次性猜整页
- 优先读取设计工具右侧面板、DOM、可访问性信息和 CoDesign metadata JSON 中的真实样式值
- 保持现有业务逻辑、数据流、路由、接口调用和权限逻辑不变
- 在最终结果中说明样式来源、资产处理方式、还原比例和剩余差异

## 仓库结构

```text
.
+-- README.md
+-- LICENSE
+-- commands/
|   +-- DesignDraft.md
+-- skills/
    +-- design-draft/
        +-- SKILL.md
        +-- agents/
        |   +-- openai.yaml
        +-- references/
            +-- asset-rules.md
            +-- codesign.md
            +-- content-region.md
            +-- mockplus.md
            +-- pixel-restore-rules.md
            +-- platform-detection.md
            +-- reporting.md
            +-- restoration-workflow.md
            +-- style-conversion.md
            +-- unit-splitting.md
```

其中：

- `skills/design-draft/SKILL.md` 是 Skill 的主入口说明
- `skills/design-draft/agents/openai.yaml` 是面向支持该格式的 agent host 的元信息
- `skills/design-draft/references/` 存放还原流程、平台规则、像素还原、资产处理和报告要求等详细参考
- `commands/DesignDraft.md` 是可选的 slash command 入口，不是 Codex 安装 Skill 的必需文件

## 在 Codex 中安装

在 Codex 中输入：

```text
安装这个 Codex Skill：
https://github.com/Bohaohao/design-draft-codex-skill/tree/main/skills/design-draft
```

安装完成后，重启 Codex，让它重新发现新安装的 Skill。

## 手动安装

如果当前环境不支持自动安装，可以手动复制 Skill 目录：

```text
skills/design-draft -> $CODEX_HOME/skills/design-draft
```

如果没有设置 `CODEX_HOME`，Codex 通常使用：

```text
~/.codex/skills/design-draft
```

复制完成后同样需要重启 Codex。

## 使用方式

先在 Chrome 中打开目标 Mockplus 或 CoDesign 设计稿页面，然后在 Codex 中描述要还原的页面和目标文件。

示例：

```text
使用 design-draft skill，把当前 CoDesign 页面还原到 src/views/example/index.vue。
```

也可以在支持 slash command 的环境中使用：

```text
/DesignDraft src/views/example/index.vue
```

Skill 会在以下情况向用户确认：

- 目标设计稿标签页不明确
- 目标 Vue 文件没有指定
- 业务内容区域边界不清楚
- 设计稿中的元素无法通过工具读取到足够信息
- 多个切图或资产候选都可能匹配

## 运行前提

使用这个 Skill 需要：

1. 当前 agent host 可以访问本地项目文件
2. Chrome 中打开了目标设计稿页面
3. `chrome-devtools-mcp` 可用
4. 当前工作区允许写入

如果缺少必要条件，Skill 会先报告缺失项，而不是直接猜测或写代码。

## 图像理解能力边界

这个 Skill 不要求模型一定具备图像理解能力。

如果宿主明确支持图像输入，并且当前 agent 能安全理解图片附件，Skill 会设置：

```text
imageUnderstanding = available
```

此时图像理解可以用于：

- 辅助确认设计稿区域边界
- 对还原区域做视觉分类
- 识别可点击模块、控件、图标、文本块和安全点击点
- 辅助比较切图候选
- 做最终视觉检查

但图像理解不能替代真实数据来源。以下信息仍必须优先来自 Chrome MCP 可读内容、设计工具 metadata 或用户确认：

- 右侧样式面板中的 CSS
- CoDesign metadata JSON
- DOM 文本
- accessibility snapshot
- selected node metadata
- 元素坐标、尺寸、约束和父子布局关系
- 用户已确认的业务边界

如果宿主不支持图像理解，或能力未知，Skill 会设置：

```text
imageUnderstanding = unavailable
```

此时必须跳过截图 OCR、截图分类、视觉比较和截图估算，但不能因此停止任务。Skill 会继续使用：

- CoDesign metadata JSON
- 右侧属性、样式、inspect 或 CSS 面板
- 图层名称和层级
- DOM 文本和 accessibility 信息
- 元素角色、尺寸、坐标和组件元数据
- 项目现有实现约定
- 用户确认

关键原则：

```text
metadata / panel 优先，图像理解只做辅助；不可用时不阻塞；估算值必须明确标注。
```

## CoDesign 还原原则

在 CoDesign 中，Skill 会优先尝试读取 `design/screensMap` 和 `meta_url` 对应的 metadata JSON。

metadata JSON 中的这些字段会被视为真实机器可读来源：

- `rect`
- `realRect`
- `css`
- `fragments`
- `fills`
- `borders`
- `effects`
- `radius`
- 父子节点关系

如果 metadata JSON 可用，不能用截图猜测覆盖这些值。截图最多用于发现目标、辅助点击和最终对比。

CoDesign 右侧 CSS 面板的读取会优先使用 DOM 提取方案，而不是截图 OCR。

## 资产和切图处理

遇到图片、logo、icon、背景图、复杂图形或导出切图时，处理顺序是：

1. 检查选中元素是否暴露下载或复制切图能力
2. 如果存在切图，下载或导出
3. 如果当前页面没有切图但元素明显像切图，尝试定位名为 `切图` 的设计稿页面
4. 在切图页中查找匹配资产
5. 下载到项目的 `src/assets/images`
6. 在 Vue 文件或组件中 import 使用
7. 如果项目内已有同样资产，优先复用
8. 如果无法取得资产，使用同尺寸占位并在结果中说明差异

不会凭空编造品牌图、产品图或误导性的近似 icon。

## 输出报告

执行结束时，Skill 应报告：

- 使用的设计稿来源和业务区域
- 创建或修改的文件
- 已还原的模块
- 样式提取是否完整
- 是否使用 CoDesign metadata JSON
- 关键值分别来自 `metadata`、`panel`、`layer`、`dom`、`script`、`project`、`user` 还是 `estimated`
- 资产是下载、复用、近似、占位还是未使用
- 估算还原比例
- 剩余差异、无法验证的状态或未解决歧义

如果图像理解不可用，不能报告视觉估算值，只能报告未解决项或请求用户确认。

## 维护说明

- 可安装的 Skill 包应始终放在 `skills/design-draft`
- 面向人的说明文档放在仓库根目录，不要塞进 Skill 包内部
- 复杂流程和平台细节放在 `skills/design-draft/references`
- 修改 `SKILL.md` 或 references 后，建议用一个新的 Codex 会话测试触发和执行效果
- 不要把本地临时截图、调试 JSON、`node_modules` 或项目构建产物提交到仓库

## License

MIT
