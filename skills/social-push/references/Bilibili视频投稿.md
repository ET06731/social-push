## 发布 Bilibili 视频投稿 workflow

> 适用于通过 Bilibili 创作中心上传视频稿件。B 站投稿页会随账号权限、分区、活动和审核规则变化，所有 ref 都必须以实时 `snapshot -i` 为准。

1. 打开 Bilibili 创作中心投稿页：`agent-browser --auto-connect open "https://member.bilibili.com/platform/upload/video/frame"`
2. 等待页面加载：`agent-browser --auto-connect wait --load networkidle`
3. 查看交互元素：`agent-browser --auto-connect snapshot -i`
4. 如果未登录，暂停自动化，让用户手动登录并完成必要验证；登录完成后重新执行 `agent-browser --auto-connect snapshot -i`
5. 上传视频文件：
   - 优先使用文件输入框：`agent-browser --auto-connect upload "input[type='file']" "{视频路径}"`
   - 如果页面只显示上传按钮，先点击上传入口，再根据 snapshot 找到 file input 或系统文件选择流程
6. 等待视频上传和转码校验开始：`agent-browser --auto-connect wait 3000`
7. 查看上传状态：`agent-browser --auto-connect snapshot -i`
8. 填写标题：`agent-browser --auto-connect fill @标题输入框 "{标题}"`
9. 选择分区：
   - `agent-browser --auto-connect click @分区选择入口`
   - `agent-browser --auto-connect snapshot -i`
   - `agent-browser --auto-connect click @目标一级分区`
   - `agent-browser --auto-connect snapshot -i`
   - `agent-browser --auto-connect click @目标二级分区`
10. 填写标签：
    - `agent-browser --auto-connect fill @标签输入框 "{标签1}"`
    - `agent-browser --auto-connect press Enter`
    - 多个标签重复输入并回车确认
11. 填写简介：`agent-browser --auto-connect fill @简介输入框 "{简介}"`
12. （可选）上传封面：`agent-browser --auto-connect upload "input[type='file'][accept*='image']" "{封面路径}"`
13. （可选）按用户要求设置转载/自制、定时发布、合集、互动声明、字幕、充电、声明与活动等字段；每次打开弹窗或下拉框后都要重新 `snapshot -i`
14. 查看当前状态：`agent-browser --auto-connect snapshot -i`
15. 等待上传完成或至少确认页面已进入可编辑稿件信息状态
16. 停留在投稿确认前页面，提示用户自行检查标题、分区、标签、封面、版权声明和审核规则；禁止自动点击 `立即投稿` / `发布` / `提交稿件` 按钮

## 元素参考（示例，实际以 snapshot 为准）

| 元素 | 功能 | 说明 |
|------|------|------|
| `input[type='file']` | 视频上传 | 上传本地视频文件 |
| `textbox "标题"` | 标题输入框 | 稿件标题 |
| `button` / `combobox` 分区入口 | 分区选择 | 通常需要一级、二级分区 |
| `textbox "标签"` | 标签输入框 | 输入后回车确认 |
| `textbox "简介"` / `textarea` | 简介输入框 | 视频描述 |
| `input[type='file'][accept*='image']` | 封面上传 | 上传本地封面图 |
| `button "立即投稿"` / `button "发布"` | 最终投稿 | 禁止自动点击 |

## 注意事项

- B 站上传和转码耗时较长，等待时可以重复执行 `snapshot -i` 查看状态
- 标题、分区、标签通常是必填项，分区会影响可选字段和审核规则
- 版权声明、转载来源、AI 生成内容声明等字段必须按用户真实情况填写
- 如果出现实名认证、手机号验证、创作中心协议或版权声明弹窗，停止自动化并提示用户手动处理
- 最终只能停留在待投稿状态，不自动提交稿件
