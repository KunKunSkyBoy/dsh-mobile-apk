# known-gaps.md — 0.14.5 当前缺口与验收入口

> 更新：2026-10-07。目标 0.14.5；0.14.4 是旧包与反馈基线。源码有改动不等于新包已验收。
> 当前自主开发与模拟器验收已获用户授权；远程推送、PR、合并和发版仍需最终授权。历史版本的“外测由别人做”不适用于本轮。

## 当前入口

- 协调仓 `docs/REFACTOR-2026-10-05.md` 是本轮唯一活动计划、Issue Ledger 与事实账；子仓地图从 [AGENTS.md](../../AGENTS.md) 进入。
- 恢复与用户数据：[PERSISTENT-DATA.md](PERSISTENT-DATA.md)；协调仓 `docs/RECOVERY-ARCH-0.14.5.md`。当前实现以源码和冻结后的证据为准。
- 三层验收：[emulator-test-protocol.md](emulator-test-protocol.md)。旧包、热推或仅静态审计不能代替最终双 ABI 包的代码/CDP/ADB 用户层验收。

## 本轮仍需收口

| 项 | 当前事实 | 必须取得的结论 |
|---|---|---|
| Hard / Soft 插件状态 | 可信 factory identity、四态与真实冷启动健康正在实施；live 用户 patch 不能成为 Hard 来源 | 冻结后统一测试；Disabled 官方条目不能阻止健康；隔离不丢源码/config；退出 Safe Mode 恢复用户配置 |
| ONLINE 事务与健康 | 完整 tar 和 usr-only 兼容、只交换 usr、Hard sidecar 与 probation/commit/rollback、真实完成态正在修订 | 最终新包验证不改用户 HOME；仅 HTTP 活着不得提前提交；失败后可恢复 |
| 回撤节流纪元 | `.undo-auto-done` schema 2 仅抑制相同安装指纹 + 回滚前失败 patch digest，成功回滚后原子写入；身份变化重开观察窗，旧数字时间戳在有效期内保守抑制并记诊断；显式用户重启在确认本壳子进程停止后清 marker，自动重试不清。纯逻辑与源码接线测试已补，仍待本批 Gradle fresh 验证 | 同一失败配置防循环，新安装/新配置不继承旧事件；marker 写入只发生在成功整份恢复后 |
| 许可证 | Shizuku api/provider/aidl/shared 13.1.5 官方 POM 均为 MIT；已补固定许可文本、矩阵、notices 生成及版本/文本反例 | 最终 APK assets/licenses 内含 MIT 全文与 SDK notices；当前源码门禁通过不等于实包已确认 |
| MiMo 与配置保留 | 旧基线真实 wire on/off 成功；虚拟屏目标达成但旧任务 13 次工具调用超出事先 10 次限制，整例未通过 | 新正式包自主任务预先约定可行预算并按真实事件计数；保留既有配置/会话/凭据 |
| UI / 范围 / 通知 / 浏览器 | 有旧版和阶段证据；最终当前树尚未完成三层回归 | 两个模拟器横竖屏、透明度、拒绝面、文件选择、后台任务和旧会话均按同一产物复验 |
| 真机 | MuMu x86_64 是开发首验；arm64 真机是发布前补充 | 华为 WebView identity/安装反馈不能以模拟器替代真机证据 |
| CI 面两个已死 workflow 条目 | `Snapshot Build`（本地已 `git rm`，未推送）与 `ADB runtime patch`（其分支 `contrib/adb-runtime-patch` 已不存在）仍出现在 Actions 列表，均已 `gh workflow disable`；REST 无删除 workflow 的接口 | push 后 `Snapshot Build` 自动消失；`ADB runtime patch` 需在默认分支上不存在该文件，当前仅能保持 disabled |
| APK 更新兜底的例外：`runUpdate` 无超时兜底 | 0.14.6 新增的 APK 更新路径（`ApkUpdateController`）有完整兜底（`armWatchdog`/`disarmWatchdog`/`onTimeout`/`timeoutReason`/`release`）；**但快照联调路径 `runUpdate`（`EngineStartFlow.kt:1143`）没有任何定时器**，回执守卫在 `:1150`，代次作废即静默丢弃回执、不落终态 | 已裁定**本轮不修**（三条理由：① 入口是 `MainActivity.kt:475-476` 的 `ACTION_UPDATE`（`:197`，manifest 未声明），属 **adb-only 联调路径、普通用户不可达**；② 默认配置 `DEFAULT_MANIFEST_URL` 为空 ⇒ `UpdateManager` 立刻回 `NotConfigured` ⇒ 实际快速收敛、暴露不到；③ 本轮已为「下载 APK 更新」重新出包，再动 liveness 路径的回归风险不抵收益）。**如实登记的结论**：与用户报的「更新转圈不收敛」**同机制**，但**非用户可达**；若将来有人用 `overrideManifestUrl` 打开联调且下载卡住，才会以同形暴露 |
| `check-code-map.mjs` 锚点只验行号存在性、不验语义 | 锚点判定只校验「行号 ∈ [1, 文件行数]」（`scripts/check-code-map.mjs:235-242`），**不校验该行内容是否仍是文档声称的符号**。源码前移后地图会**静默漂移**而门禁仍 `exit 0`。实测（2026-10-11，只读探针）：1633 个锚点中带可提取预期符号者 **667 个（40.8%）**，可判读的 495 个里约 **53%（263 个）已漂移**；手工抽查 4 例 **4/4 真漂移** —— `AndroidBridge.kt` 的 `jsString` 文档写 `:423`（现 `:675`）、`BrowserHost.kt` 的 `onMain` 写 `:187`（现 `:386`）、`UpdateManager.kt:69` 写 `renameTo`（该行现为 `IllegalStateException`）、`SnapshotFs.kt:83` 写 `move`（现 `:74`） | 已裁定**本轮不改门禁**（两条理由：① 它是发布链门禁，改它需自证，而本轮已多次代码改动 + 第三次出包，风险叠加；② **未调参就启用比不加更坏** —— 会训练人忽略门禁）。**建议实现方向**：① 文档侧先给锚点补预期符号（纯文档、零门禁风险），把可校验覆盖率从 40.8% 往上提；② 门禁侧再加「`line ± WINDOW` 内出现预期符号」的校验 + **反例自检**（「符号移走能被发现」与「符号在窗口内不得判红」），WINDOW 按基线调参至误报为 0 再启用。**注意**：朴素窗口校验误报率高（59.2% 的锚点提取不到预期符号；裸文件名锚点如 `MainActivity.kt:815` 按仓内约定相对 `app/src/main/java/com/dsharnessmobile/shell/`），**不要在未调参时直接启用** |

## 历史事项不自动关闭

旧登记的浏览器自动落位、看门狗恢复入口、API<30 截图回落、多 ROM 限制、动效、滚动条、低置信度依赖判据等，保留当时证据。应按当前源码逐项分类为已改、仍存在或仅特定设备待验；不得把旧“未修”直接当当前事实，也不得把归档当完成。

完整原文在 [历史缺口登记](../archive/known-gaps-pre-0.14.5-2026-10-07.md)。当时版本结论、授权和停止点均不覆盖当前本轮要求。
