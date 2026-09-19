# 我们的更改说明

这份文档记录本次协作中对 `YuyanIme` 及 `yuyansdk` 自己维护线做出的修改。后续每次修改都在顶部追加记录，并同时写明提交、构建和实机验证状态。

状态约定：

- `已验证`：代码、构建和实机行为均已确认。
- `待验证`：代码和构建已完成，但还没有完成对应设备上的行为测试。
- 上游 issue/PR 只作为参考，不代表直接合并了上游代码。

## 2026-09-19

### 维护这份更改说明

- 新增本文件，作为项目内持续维护的修改记录。
- Android 16 主力机：realme `RMX5200`，API 36。
- Android 14 主力机：realme `RMX3888`，API 34。
- 两台设备均已安装 `20260918.19` Release；物理键盘行为尚未验证，状态为 `待验证`。

## 2026-09-18

### 物理键盘和候选栏

- SDK 提交：`8a3e5f0`，父仓库同步提交：`c69781f`。
- Del 改用 `deleteSurroundingText(1)`，避免合成物理按键 down/up 导致一次删除两个字符。
- 修正 `ImeService` 中 `onKeyDown()`、`onKeyUp()` 的错误 fallback，确保 Down/Up 使用对应的父类方法。
- 修正重复键、Shift/Meta/Ctrl 组合键的释放事件转发。
- Android 13+ 的 Back 成对交给系统返回仲裁；Android 12 及以下保留输入法主动隐藏逻辑，避免产生孤儿 UP 和系统返回失效。
- CandidateView 增加物理键盘 Back、Tab、上/下方向键处理，并让方向键、英文功能键的 Down/Up 成对。
- Ctrl+Space 优先切换中英文，不再先提交空格或当前候选词；软键盘和物理键盘路径均处理。
- 物理数字键只在存在对应候选时选择 1-9；无候选、无效序号和 0 正常转发给编辑器。
- 候选栏在旋转后重新计算宽度。
- 候选提交后清理旧候选状态，避免剪贴板/联想候选残留。
- 移除调用方重复的花漾字转换，统一由 `ImeService.commitText()` 转换一次。
- 增加输入连接、输入法 View 和 Insets 计算的空安全，降低输入法隐藏或重绑时崩溃风险。
- 硬键盘检测同时考虑 `hardKeyboardHidden` 状态。
- 未改变“物理键盘显示软键盘”设置的既有语义：开启时显示完整软键盘，关闭时显示候选/输入栏。

构建状态：

- Offline Debug/Release 构建成功。
- Android 16、Android 14 均已安装。
- Del、Back、数字键、Ctrl+Space、候选栏切换等行为仍待实机验证。

### 上游参考

- [yuyansdk PR #25](https://github.com/gurecn/yuyansdk/pull/25)：确认 Android 13+ Back 消费、未初始化 View 和空输入连接风险。
- [物理键盘 Ctrl 组合键提交](https://github.com/gurecn/yuyansdk/commit/d2667589e5f2eee358051da36a3fdef36330e6fc)：确认 Ctrl+Space 是特殊组合键。
- [物理键盘显示软键盘实现](https://github.com/gurecn/yuyansdk/commit/1374c1ef814ddfe8eb5c1ea9214174181217e035)：确认开启设置时显示完整软键盘属于原设计。

## 2026-09-18

### Android 13+ 返回键初步修复

- SDK 提交：`6ae1545`，父仓库同步提交：`0a4bb97`。
- 修复输入法在 Android 13+ 上无法正常隐藏的问题，并增加 Android 新版本设备回归版本。
- 后续在物理键盘审计中按照上游 PR #25 将 Back 调整为 Android 13+ 完整交给系统处理，最终逻辑见 `8a3e5f0`。

## 2026-09-17

### 双拼回车提交原始输入

- SDK 提交：`1ee595a`，父仓库同步提交：`d15fa48`。
- Rime/Kernel 暴露原始组合输入。
- `DecodingInfo.composingStrForEnter` 在双拼方案回车时使用原始输入，避免把内部编码转换成错误的拼音文本。
- `InputView` 和 `CandidateView` 的回车提交统一使用该值。
- 已验证：输入 `wj` 后回车，上屏内容为 `wj`。

## 仓库和构建维护

- 当前使用线统一为父仓库和 SDK 的 `master`。
- 旧线保留为 `master-old`，用于回溯，不删除旧历史。
- SDK 子模块改为指向自有仓库 `NBLJSBDK/yuyansdk`，父仓库 `.gitmodules` 已同步。
- Release 签名改为从本地属性读取，不再把签名路径和密码写死在构建脚本中。
- App 启动入口显式初始化 `CrashHandler` 和 SDK `Launcher`。
- APK 归档到本地 `assets/release`、`assets/debug`，通过本地 Git exclude 排除，不进入 Git。
- 构建使用临时 JDK 17 和 Gradle 8.4，不修改项目编译规则。

## 版本归档

APK 文件保存在项目本地 `assets` 目录；对应 `BUILD-*.txt` 保存源码提交、SHA-256 和验证状态。

| 版本 | 内容 | SDK 提交 | Release SHA-256 | Debug SHA-256 | 状态 |
| --- | --- | --- | --- | --- | --- |
| `2026091718` | 双拼回车原始输入 | `1ee595a` | `773032aae07c4acfd9cdd631a174d3e12f27b6afc908d3446278c0f85939e3cb` | `8782bf6d7a2413e0b6947404b1a0a9a8bcbe1d95173dddc253fb28e059ba347e` | 已验证 |
| `2026091815` | Android 13+ 返回键修复 | `6ae1545` | `a9a917d0d40ea984198e55560a3ae34b3f887b460e4940766c5d58b1046e1687` | `29a812d1d7dedaf8d77f9a350d3a59c0ae779234e7d17bf7f8c469248119841c` | 已安装 |
| `2026091819` | 物理键盘按键与候选栏修复 | `8a3e5f0` | `d82dd278b24bd945f32fc12ff0ddb236e9a4950ff4fac5e9c4e1eaec70b944c8` | `9db218ad99417e209e3221e2e3367de14f71872a994753dfdfdbf2fe8f95bfc9` | 待验证 |

## 本地已有基线

以下提交在本次协作开始前已经存在，作为当前 SDK 基线使用，没有在本记录中冒充为本轮新增修改：

- `57ec812`：返回手势崩溃与卡顿修复。
- `6be4a85`：构建时间显示修复。
- `7839d2d`：滑动光标越界失焦修复。
- `703c523`：同步主线键盘符号配置。
