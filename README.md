# Video Workbench

Video Workbench V1 是一个给个人使用的本地视频制片工作台。它使用单个 `index.html` 运行，不接任何 API，不做后端，项目数据保存在浏览器 localStorage。

## 打开方式

直接使用 Edge、Chrome 或其他现代浏览器打开 `index.html`。

如果浏览器限制了 `file://` 页面使用 localStorage，可在当前目录启动一个简单的本地静态服务器：

```powershell
python -m http.server 8000
```

然后访问：

```text
http://localhost:8000
```

## 功能范围

- 选择观点解释模式或主观表达模式。
- 输入原始稿件，并按空行和句末标点初步拆分为 Beat。
- 新增、编辑、复制、删除和排序 Beat。
- 编辑 Beat 的 `mode`、`text`、`duration`、`visual_kind`、`visual_brief`、`prompt`、`chart_data`、`audio_hint`、`source` 和 `rights`。
- 选择口播、实拍空镜、图片、屏幕录制、文字卡、图表、动画、资料影像、抽象意象、混合素材和无画面。
- 记录素材来源和使用权利。
- 导出 Markdown。
- 导出带 UTF-8 BOM 的 CSV，便于 Excel 正确显示中文。
- 导入和导出项目 JSON。
- 使用 localStorage 自动保存。
- 首次打开自动创建示例项目。

## 数据备份

项目数据只保存在当前浏览器和当前访问地址对应的 localStorage 中。

完整备份方法：

1. 打开需要备份的项目。
2. 点击“项目 JSON”。
3. 保存下载的 JSON 文件。

恢复方法：

1. 点击“导入 JSON”。
2. 选择之前导出的项目 JSON。
3. 导入成功后，项目会作为新项目加入项目库并自动打开。

Markdown 和 CSV 适合阅读、审阅和表格处理，不包含完整恢复项目所需的全部结构和 id，不能替代项目 JSON 备份。

## 已知限制

- localStorage 容量通常只有约 5 MB，不适合保存视频、音频或图片文件。
- 清理浏览器站点数据、更换浏览器或移动 `index.html` 的访问地址后，原有数据可能无法读取。
- 第一版不保存媒体文件，也不提供视频预览、时间线、自动剪辑或视频渲染。
- 稿件拆分和 duration 都是本地规则估算，不理解语义。
- `chart_data` 只手工保存 JSON，不自动生成图表。
- `prompt` 只保存和复制，不执行提示词路由，也不调用任何生成服务。
- JSON 导入只支持 `schemaVersion: 1`。
- 没有账号、云同步、协作、自动配音、自动字幕或剪映工程控制。
- 没有撤销历史，删除数据前应先导出项目 JSON 备份。
