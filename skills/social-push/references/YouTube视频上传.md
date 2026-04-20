## 发布 YouTube 视频上传 workflow

> 适用于通过 YouTube Studio 上传视频。YouTube 上传流程通常包含多个弹窗步骤，所有 ref 都必须以实时 `snapshot -i` 为准。

1. 打开 YouTube Studio 上传入口：`agent-browser --auto-connect open "https://studio.youtube.com/"`
2. 等待页面加载：`agent-browser --auto-connect wait --load networkidle`
3. 查看交互元素：`agent-browser --auto-connect snapshot -i`
4. 如果未登录，暂停自动化，让用户手动登录 Google 账号并完成必要验证；登录完成后重新执行 `agent-browser --auto-connect snapshot -i`
   - 如果浏览器里已经有已登录的 Studio 标签，但 agent-browser 连接到了登录跳转页，先用 `http://127.0.0.1:9222/json/list` 找到 `studio.youtube.com` 页面对应的 `webSocketDebuggerUrl`，再执行：`agent-browser connect "{webSocketDebuggerUrl}"`
5. 点击 `Create` / `创建` 按钮，再选择 `Upload videos` / `上传视频`：
   - `agent-browser --auto-connect click @创建按钮`
   - `agent-browser --auto-connect snapshot -i`
   - `agent-browser --auto-connect click @上传视频菜单项`
   - 也可以直接打开频道内容上传入口：`agent-browser open "https://studio.youtube.com/channel/{频道ID}/videos/upload"`，再点击页面里的 `上传视频`
6. 查看上传弹窗：`agent-browser --auto-connect snapshot -i`
7. 上传视频文件：
   - 优先使用文件输入框：`agent-browser --auto-connect upload "input[type='file']" "{视频路径}"`
   - 上传后等待详情页出现：`agent-browser --auto-connect wait 3000`
8. 查看详情页元素：`agent-browser --auto-connect snapshot -i`
9. 填写标题：`agent-browser --auto-connect fill @标题输入框 "{标题}"`
   - 如果标题没有被清空而是追加到原文件名后，使用：`click @标题输入框` -> `press "Control+a"` -> `keyboard inserttext "{标题}"`
10. 填写说明：`agent-browser --auto-connect fill @说明输入框 "{简介}"`
11. （可选）上传缩略图：`agent-browser --auto-connect upload "input[type='file'][accept*='image']" "{缩略图路径}"`
12. 选择播放列表（如用户要求）：
    - 点击播放列表下拉或按钮
    - 重新 `snapshot -i`
    - 勾选目标播放列表并确认
13. 选择受众设置：
    - 必须选择 `No, it's not made for kids` / `否，不是面向儿童的内容`，或按用户明确要求选择面向儿童
    - `agent-browser --auto-connect snapshot -i`
14. （可选）填写标签：
    - 点击 `显示高级设置`
    - 如果按钮在可视区域外，先执行 `agent-browser scrollintoview @显示高级设置按钮`
    - 在 `标签` 输入框里填写逗号分隔标签，例如：`AI,Tech,人工智能,科技,简报`
    - 填写后重新 `snapshot -i`，确认页面已拆分成独立标签项
15. 点击 `Next` / `下一步` 进入后续配置页，但不要发布：
    - 视频元素页：按用户要求添加字幕、片尾画面、信息卡；否则继续下一步
    - 检查页：等待版权检查结果；若出现版权、限制或警告，停止并提示用户确认
    - 可见性页：选择 `Private` / `Unlisted` / `Public` 仅在用户明确要求时执行
16. 查看最终可见性页状态：`agent-browser --auto-connect snapshot -i`
17. 停留在最终确认页面，提示用户自行检查标题、说明、缩略图、标签、受众、版权检查和可见性；禁止自动点击 `Publish` / `发布` / `Save` / `保存` 按钮

## 元素参考（示例，实际以 snapshot 为准）

| 元素 | 功能 | 说明 |
|------|------|------|
| `button "Create"` / `button "创建"` | 创建菜单 | 打开上传入口 |
| `menuitem "Upload videos"` / `menuitem "上传视频"` | 上传视频入口 | 打开上传弹窗 |
| `input[type='file']` | 视频上传 | 上传本地视频文件 |
| `textbox` 标题字段 | 标题输入框 | YouTube Studio 详情页标题 |
| `textbox` 说明字段 | 说明输入框 | 视频 description |
| `input[type='file'][accept*='image']` | 缩略图上传 | 上传本地封面图 |
| `radio` 儿童内容选项 | 受众设置 | 必填项 |
| `button "显示高级设置"` | 高级设置 | 展开 tags、语言、许可等字段 |
| `textbox "标签"` | 标签输入框 | 逗号分隔输入会拆分为多个标签 |
| `button "Next"` / `button "下一步"` | 下一步 | 可点击进入配置页 |
| `button "Publish"` / `button "发布"` | 最终发布 | 禁止自动点击 |
| `button "Save"` / `button "保存"` | 最终保存/发布 | 禁止自动点击 |

## 注意事项

- YouTube 上传、处理和版权检查可能耗时较长，等待期间可以重复执行 `snapshot -i`
- 受众设置是必填项，不能替用户猜测儿童内容属性；没有明确说明时选择前应让用户确认
- 如果出现版权声明、账号验证、频道创建、社区准则或功能权限提示，停止自动化并提示用户手动处理
- 可见性会直接影响公开视频范围，除非用户明确指定，否则停留在最终确认页面
- 最终只能准备到待发布/待保存状态，不自动点击发布或保存
- 在 PowerShell 中使用 ref 时要加引号，例如 `agent-browser click '@e27'`，避免 `$e27` 被当成变量展开
