# Video Workbench 第一版规格

## 1. 产品定位

Video Workbench 是一个给个人使用的纯浏览器本地视频制片工作台。

第一版只处理从稿件到 Beat 编排的最小工作流：

1. 选择创作模式并输入稿件。
2. 把稿件拆成 Beat 卡片。
3. 增删改和排序 Beat。
4. 为每个 Beat 选择视觉类型。
5. 记录素材来源和使用权利。
6. 导出 Markdown 和 CSV。
7. 导入和导出项目 JSON。

第一版不接远程或第三方 API，不依赖后端，不需要账号，不包含云同步。

## 2. 明确边界

### 2.1 第一版包含

- 观点解释模式和主观表达模式。
- 新建项目、修改项目名称、删除项目。
- 稿件粘贴和按段落初步拆分为 Beat。
- Beat 卡片新增、编辑、删除、复制和排序。
- 视觉类型选择。
- 来源与权利文本记录。
- Markdown 导出。
- 带 UTF-8 BOM 的 CSV 导出，确保 Excel 打开中文不乱码。
- 单个项目的 JSON 导出。
- JSON 项目导入。
- localStorage 自动保存。
- 一个内置示例项目。

### 2.2 第一版不包含

- 提示词路由。
- LLM、图片、视频或语音 API。
- 自动配音和自动字幕。
- 自动图表生成。
- 自动剪辑和自动渲染。
- 剪映工程控制。
- 后端、远程数据库、账号登录和跨设备同步。
- 时间线编辑、媒体播放和视频导出。
- 多人协作、评论审批和发布流程。

## 3. 技术约束

- 使用单个 `index.html`。
- 使用原生 HTML、CSS 和 JavaScript。
- 不引入复杂框架，不依赖构建工具，不加载外部资源。
- 所有项目数据只保存在浏览器 localStorage。
- 页面刷新后自动恢复最近一次项目状态。
- 不发送任何网络请求。
- 导出文件使用浏览器原生下载能力。
- 导入 JSON 使用浏览器本地文件选择能力。

## 4. 核心数据

### 4.1 LocalState 本地状态

localStorage 保存一个根对象。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| schemaVersion | 整数 | 是 | 第一版固定为 1 |
| projects | Project 列表 | 是 | 所有项目 |
| activeProjectId | 字符串 | 否 | 最近打开的项目 |
| ui | UIState | 是 | 必要的界面状态 |

建议使用一个固定的 localStorage 键保存根对象。业务数据不得散落到多个不可恢复的键中。

### 4.2 Project 项目

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | 字符串 | 是 | 全局唯一标识 |
| name | 字符串 | 是 | 项目名称 |
| mode | 枚举 | 是 | explainer、expressive |
| manuscript | 字符串 | 是 | 原始稿件 |
| beats | Beat 列表 | 是 | 按顺序排列 |
| createdAt | 时间 | 是 | 创建时间 |
| updatedAt | 时间 | 是 | 最近修改时间 |

### 4.3 Beat 核心对象

Beat 是唯一的核心创作对象。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | 字符串 | 是 | 系统生成的唯一标识 |
| mode | 枚举 | 是 | explainer、expressive |
| text | 字符串 | 是 | Beat 正文 |
| duration | 数值 | 是 | 预期时长，单位秒，必须大于 0 |
| visual_kind | 枚举 | 是 | 视觉类型 |
| visual_brief | 字符串 | 是 | 画面说明；none 类型可以为空 |
| prompt | 字符串 | 否 | 本地保存的生成或编辑提示 |
| chart_data | 对象或空值 | 否 | 手工填写或粘贴的结构化图表数据 |
| audio_hint | 字符串 | 否 | 音乐、音效、语气、停顿或静音提示 |
| source | 字符串 | 否 | 素材来源；多个来源按行记录 |
| rights | 字符串 | 否 | 使用权利说明；默认待确认 |

第一版只使用这些字段。系统 id 只在项目 JSON 中保留，不在卡片主界面突出显示。

### 4.4 VisualKind 视觉类型

| 值 | 界面名称 |
| --- | --- |
| talking_head | 口播 |
| b_roll | 实拍空镜 |
| image | 图片 |
| screen_recording | 屏幕录制 |
| text_card | 文字卡 |
| chart | 图表 |
| animation | 动画 |
| archive | 资料影像 |
| abstract | 抽象意象 |
| mixed | 混合素材 |
| none | 无画面 |

### 4.5 ChartData 图表数据

第一版不自动生成图表，只允许在 Beat 中手工输入 JSON。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| kind | 字符串 | 是 | bar、line、pie、scatter、table |
| title | 字符串 | 是 | 图表标题 |
| categories | 字符串列表 | 否 | 分类或横轴标签 |
| series | 列表 | 是 | 每项包含 name 和 values |
| unit | 字符串 | 否 | 数值单位 |
| highlight | 字符串 | 否 | 强调的数据点 |
| note | 字符串 | 否 | 解读提示 |
| source_note | 字符串 | 否 | 图表来源 |

输入内容必须是合法 JSON。保存失败时保留原输入并显示字段级错误。

### 4.6 UIState 界面状态

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| activeView | 枚举 | 是 | library、workspace |
| activeProjectId | 字符串 | 否 | 当前项目 |
| lastSaveStatus | 枚举 | 是 | saved、saving、error |

### 4.7 示例项目

首次打开且 localStorage 中没有项目时，创建一个示例项目。

示例项目使用观点解释模式，包含至少三个 Beat，并覆盖：

- 不同视觉类型。
- duration 和 audio_hint。
- source 和 rights。
- 一个手工 chart_data。

用户删除示例项目后，可以通过“恢复示例项目”重新创建。恢复操作生成新的项目 id，不恢复已删除项目的修改历史。

## 5. 页面结构

### 5.1 项目库

项目库显示：

- 产品名称和一句功能说明。
- 新建项目按钮。
- 导入 JSON 按钮。
- 恢复示例项目按钮。
- 已有项目卡片。

每个项目卡片显示：

- 项目名称。
- 模式。
- Beat 数量。
- 预计总时长。
- 最近修改时间。

项目卡片操作：

- 打开项目。
- 修改名称。
- 导出项目 JSON。
- 删除项目。

### 5.2 新建项目

新建项目区域包含：

- 项目名称。
- 模式选择：观点解释、主观表达。
- 稿件文本区。
- 自动拆分 Beat 开关，默认开启。
- 创建项目按钮。

创建行为：

1. 输入项目名称和稿件。
2. 选择模式。
3. 点击创建项目。
4. 按空行和句末标点拆分稿件。
5. 每一段生成一个 Beat。
6. duration 根据文字数量估算，初始值不小于 2 秒。
7. 新 Beat 的 mode 沿用项目模式。
8. visual_kind 默认使用 b_roll。
9. rights 默认使用“待确认”。
10. 创建后进入工作台。

如果关闭自动拆分，整份稿件只生成一个 Beat。

### 5.3 工作台

工作台由以下区域组成：

- 顶部栏：返回项目库、项目名称、保存状态、模式、导入 JSON、导出 JSON。
- 工具条：新增 Beat、导出 Markdown、导出 CSV、恢复示例项目。
- 摘要条：Beat 数量、预计总时长、待确认权利数量、缺少 source 数量。
- 主体：Beat 卡片列表。
- 原始稿件抽屉：允许查看和重新拆分稿件。

桌面端使用居中的单列卡片流。卡片宽度保持稳定，窗口变窄时改为全宽。

### 5.4 Beat 卡片

每张卡片显示：

- 序号和拖动把手。
- mode。
- text。
- duration。
- visual_kind。
- visual_brief。
- audio_hint。
- source。
- rights。
- prompt。
- chart_data。

卡片折叠状态显示：

- 序号。
- text 摘要。
- duration。
- visual_kind。
- source 是否有内容。
- rights 状态。

卡片操作：

- 展开或折叠。
- 编辑任意字段。
- 复制 Beat。
- 删除 Beat。
- 上移。
- 下移。
- 拖动排序。

删除最后一个 Beat 时允许删除，但显示空状态和新增入口。

## 6. Beat 交互

### 6.1 新增

- 点击“新增 Beat”。
- 新 Beat 插入列表末尾并自动展开。
- 默认 text 为空，duration 为 4 秒。
- mode 使用项目模式。
- visual_kind 使用 b_roll。
- rights 使用“待确认”。

### 6.2 编辑

- 文本和数字字段修改后立即进入待保存状态。
- 500 毫秒内没有继续输入时写入 localStorage。
- 数字字段非法时显示错误。
- chart_data 非法 JSON 时显示错误。
- 离开字段不丢失未提交内容。

### 6.3 删除

- 点击删除时使用页面内确认条。
- 确认后删除 Beat。
- 取消后保持数据不变。

### 6.4 排序

- 卡片支持拖动排序。
- 同时提供上移和下移按钮。
- 排序后立即保存。
- 序号随列表顺序重新计算。

### 6.5 视觉类型

- 使用下拉菜单选择。
- 选择 none 时允许 visual_brief 为空。
- 选择 chart 时展开 chart_data 输入区。
- 其他类型仍允许手工填写 chart_data，但不强制。

### 6.6 来源和权利

- source 和 rights 使用多行文本。
- 多个来源和多个权利说明按行记录。
- 摘要条统计为空或包含“待确认”的 Beat。
- 第一版只记录，不联网验证来源，也不判断版权风险。

### 6.7 提示和声音

- prompt 只作为本地文本保存。
- 界面只提供编辑和复制，不提供运行、路由或生成按钮。
- audio_hint 只作为本地文本保存，不播放、不合成音频。

## 7. localStorage 持久化

- 根对象写入固定的 localStorage 键。
- 每次项目、Beat 或项目名称变化后自动保存。
- 保存成功时显示“已保存”。
- 保存中显示“保存中”。
- 写入失败或超过容量时显示“保存失败”。
- 保存失败不得覆盖上一次成功保存的数据。
- 项目 JSON 导入成功后立即写入 localStorage。
- 页面加载时校验 schemaVersion。
- 数据损坏时显示恢复入口，不自动清空全部项目。

## 8. 导出

### 8.1 Markdown

Markdown 文件包含：

- 项目名称。
- 项目模式。
- 预计总时长。
- Beat 总数。
- 每个 Beat 的序号、mode、text、duration、visual_kind、visual_brief、prompt、chart_data、audio_hint、source 和 rights。

空字段使用“未填写”，不得输出 undefined 或 null。

### 8.2 CSV

CSV 必须带 UTF-8 BOM。

列顺序固定为：

1. index
2. mode
3. text
4. duration
5. visual_kind
6. visual_brief
7. prompt
8. chart_data
9. audio_hint
10. source
11. rights

规则：

- 第一行是列名。
- 所有字段使用双引号包裹。
- 字段内的双引号转义为两个双引号。
- 字段内的换行保留。
- 文件名包含项目名和日期。
- 目标软件为 Excel 时中文不能乱码。

### 8.3 项目 JSON

导出的 JSON 包含：

- schemaVersion。
- exportedAt。
- project。

Beat 保留全部字段和 id。导出操作不修改当前项目。

## 9. 导入 JSON

导入流程：

1. 用户选择 JSON 文件。
2. 本地读取文本。
3. 解析 JSON。
4. 校验 schemaVersion、project、beats 和 Beat 必填字段。
5. 导入成功后作为新项目加入项目库。
6. 如果项目 id 已存在，生成新的项目 id。
7. 如果 Beat id 重复，生成新的 Beat id。
8. 打开导入后的项目。

导入失败时：

- 不修改现有 localStorage。
- 显示具体原因。
- 允许重新选择文件。

## 10. 编辑器规则

- text 使用多行文本框。
- duration 使用数字输入，显示秒。
- visual_kind 使用下拉菜单。
- visual_brief、prompt、audio_hint、source 和 rights 使用多行文本框。
- chart_data 使用等宽字体文本区。
- 所有输入项都有可见标签。
- 键盘焦点清晰可见。
- 删除、排序和导入导出都有可读的操作反馈。
- 小于 768 像素时单列显示，工具按钮允许换行。

## 11. 验收标准

1. 打开 `index.html` 后，首次使用会看到示例项目。
2. 能新建观点解释项目并粘贴稿件。
3. 能新建主观表达项目并粘贴稿件。
4. 开启自动拆分时，稿件会生成多个 Beat。
5. 关闭自动拆分时，稿件会生成一个 Beat。
6. 能新增、编辑、复制和删除 Beat。
7. 删除 Beat 前会出现页面内确认，取消后数据不变。
8. 能通过拖动、上移和下移调整 Beat 顺序。
9. 能选择全部规定视觉类型。
10. 选择 chart 时能填写并保存合法 chart_data。
11. 非法 chart_data 会显示错误且不会破坏已有数据。
12. 能记录 source 和 rights，并在摘要中看到缺失或待确认数量。
13. prompt 和 audio_hint 能本地保存，且不会发起网络请求。
14. Markdown 能下载并包含全部 Beat 字段。
15. CSV 能下载，包含固定列和 UTF-8 BOM。
16. CSV 用 Excel 打开时中文不乱码。
17. 项目 JSON 能下载并重新导入。
18. 导入 JSON 失败时，现有项目保持不变。
19. 刷新页面后，项目、Beat 和排序仍然存在。
20. localStorage 写入失败时有明确提示。
21. 页面运行期间不发起后端或第三方网络请求。

## 12. 已知限制

- localStorage 容量通常远小于视频文件，因此第一版不保存媒体文件。
- JSON 导入只支持本规格的 schemaVersion 1。
- 稿件拆分使用本地简单规则，不进行语义分析。
- duration 是估算值，不根据真实配音校准。
- chart_data 只保存数据，不生成图表。
- prompt 只保存文本，不发送给任何生成服务。
- 删除项目后只能通过此前导出的 JSON 恢复。
- 浏览器清理站点数据后，本地项目会丢失。
- 不同浏览器或不同站点来源之间不会自动共享数据。
