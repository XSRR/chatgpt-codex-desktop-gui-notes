# Codex Computer Use Windows sandbox 配置问题

这份笔记记录一次 Codex Computer Use 无法启动 Windows 桌面自动化 helper 的排查过程。

## 现象

在使用 Computer Use 打开记事本时，插件在初始化阶段失败，典型报错为：

```text
windows sandbox failed: spawn setup refresh
stdout_eof
```

失败发生在调用 `list_apps()` 或启动记事本之前，因此问题并不是记事本本身，而是 Windows 自动化 helper 没有成功启动。

## 根因

问题出在 `C:\Users\HP\.codex\config.toml` 的 Windows 沙盒配置。原先使用了 elevated 模式：

```toml
[windows]
sandbox = "elevated"
```

`elevated` 会让 helper 走更高权限或更隔离的启动路径。在 Windows 上，桌面自动化依赖当前用户会话、可见桌面、窗口站、权限完整性级别和辅助功能/输入注入能力保持一致。如果 helper 被放到不匹配的权限层级或隔离桌面中，它可能无法正确枚举、截图、激活或控制普通用户桌面里的应用窗口，最终在 sandbox 初始化阶段直接退出。

## 解决方法

把配置改为普通用户桌面模式：

```toml
[windows]
sandbox = "unelevated"
sandbox_private_desktop = false
```

这表示 helper 以非管理员权限运行，并且不使用私有桌面。这样它和记事本、浏览器等普通桌面应用处在同一个可见交互桌面中，Computer Use 就可以正常列出窗口、激活窗口、输入文本和读取窗口状态。

## 结论

除非明确需要控制以管理员身份运行的程序，否则 Computer Use 这类桌面自动化更适合使用：

```toml
[windows]
sandbox = "unelevated"
sandbox_private_desktop = false
```

这个配置避免了 elevated 模式下可能出现的权限层级、桌面隔离和 helper 启动失败问题。
