# BrushDigestX · 刷流汇总（修复版）

MoviePilot V3 插件：把站点刷流（BrushFlow / BrushFlowLowFreq）的新增 / 删除汇总成**一条**通知，替代每个种子一条的逐条推送。

## 为什么有这个修复版

原版「刷流汇总」(`BrushDigest`) 在 MP V3 (`>=3.1.9`) 下会崩溃，报错：

```
'list' object has no attribute 'items'
```

根因：原代码假设 `PluginDataOper().get_data_all(plugin_id)` 返回 `dict`，直接调 `.items()`；
但 V3 该接口实际返回 `list[PluginData]`（每个对象含 `.key` / `.value`），list 没有 `.items()` → 抛异常。

本插件把遍历改成兼容写法，list / dict 两种返回都能正确处理：

```python
raw = oper.get_data_all(self._source_plugin) or []
items = raw.items() if isinstance(raw, dict) else [(item.key, item.value) for item in raw]
rows = [v for k, v in items if k.startswith("task.") and k.endswith(".statistic") and isinstance(v, dict)]
```

## 安装（作为在线插件源）

1. 把本仓库推到 GitHub 的 `main` 分支。
2. MoviePilot → 设置 → 插件市场 → 插件仓库地址，加入 `https://github.com/<你的账号>/<本仓库>/`。
3. 同步插件源 → 找到「刷流汇总(修复版)」→ 安装（插件落到 `/config/plugins/`，不动容器镜像）。
4. 启用插件，数据来源选你实际在用的刷流插件（默认 `BrushFlow`）。

## 重要配置

**必须**把原刷流插件的「发送通知」关掉，否则每个种子仍会单独推送，汇总就白做了：

- `BrushFlow`（多任务版）：插件设置 → 逐个任务编辑 → 关闭该任务的「发送通知」→ 保存。
- `BrushFlowLowFreq`：插件设置 → 取消勾选「发送通知」→ 保存。

## 特性

- 自动检测本机已安装的刷流插件，下拉框选中文名即可，不用手填 ID。
- 统计口径为「距上次汇总以来的变化」，支持「无变化不推送」「显示流量」「立即汇总一次」等开关。
- 仅做汇总与通知，不刷流、不改动任何种子。
