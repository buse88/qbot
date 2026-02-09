# 彩色控制台输出

## 功能说明

为 QBot 添加了彩色控制台输出功能，让日志更加清晰易读。

## 特性

### ✅ 跨平台支持

- **Windows**: 使用 `colorama` 库自动转换 ANSI 代码
- **Linux**: 原生 ANSI 颜色代码支持
- **macOS**: 原生 ANSI 颜色代码支持
- **Docker**: 完全支持（无需额外配置）

### 🎨 颜色方案

| 颜色 | 用途 | 示例 |
|------|------|------|
| 🟢 绿色 | 成功、正常状态 | 模块加载成功、连接成功 |
| 🔴 红色 | 错误、失败 | API 调用失败、权限不足 |
| 🟡 黄色 | 警告 | 配置缺失、即将过期 |
| 🔵 蓝色 | 信息 | 当前状态、数据展示 |
| 🔷 青色 | 提示 | 操作建议、下一步 |
| 🟣 洋红色 | 特殊 | 调试信息、高亮 |

### 📦 智能降级

即使没有安装 `colorama`，程序也能正常运行：

1. **有 colorama**: 使用 colorama 库（Windows 完美支持）
2. **无 colorama**: 使用原生 ANSI 代码（Linux/Docker 原生支持）
3. **完全失败**: 降级为纯文本（仍然显示 emoji）

## 使用方法

### 基础用法

```python
from utils.colors import green, red, yellow, blue, SUCCESS, ERROR, WARNING

# 彩色文本
print(f"这是 {green('成功')} 的消息")
print(f"这是 {red('错误')} 的消息")
print(f"这是 {yellow('警告')} 的消息")

# 带符号的消息
print(f"{SUCCESS} 操作成功")
print(f"{ERROR} 操作失败")
print(f"{WARNING} 注意事项")
```

### 便捷函数

```python
from utils.colors import success, error, warning, info

print(success("模块加载成功"))
print(error("连接失败"))
print(warning("配置缺失"))
print(info("当前版本: 1.0.0"))
```

### 实际应用

```python
# main.py
from utils.colors import green, SUCCESS, ERROR

print(f"[系统] {SUCCESS} 成功连接到 QQ: {green(str(self_id))}")
print(f"[群管理] {ERROR} 撤回失败: 权限不足")

# base_module.py
from utils.colors import green, SUCCESS

print(f"[{self.name}] {SUCCESS} 模块已加载 (v{green(self.version)})")
```

## 安装

### 方式 1: 使用 requirements.txt（推荐）

```bash
pip install -r requirements.txt
```

`requirements.txt` 中已包含 `colorama==0.4.6`

### 方式 2: 手动安装

```bash
pip install colorama
```

### 方式 3: Docker（可选）

如果在 Docker 中运行，可以不安装 colorama，程序会自动使用 ANSI 代码：

```dockerfile
# Dockerfile
FROM python:3.11-slim

# 可选：安装 colorama（Windows 容器需要）
RUN pip install colorama

# 或者不安装，使用原生 ANSI 支持（Linux 容器）
```

## 测试

运行测试脚本查看效果：

```bash
python tests/test_colors.py
```

输出示例：
```
🎨 颜色输出测试
Colorama 可用: True
============================================================

基础颜色测试:
  绿色文本 - 用于成功消息
  红色文本 - 用于错误消息
  黄色文本 - 用于警告消息
  蓝色文本 - 用于信息消息

符号测试:
  ✅ 成功符号
  ❌ 错误符号
  ⚠️ 警告符号
  ℹ️ 信息符号

实际应用示例:
[系统] ✅ 成功连接到 QQ: 3121201314
[京东转换器] ✅ 模块已加载 (v1.0.0)
[群管理模块] ❌ 撤回失败: 权限不足
```

## 文件结构

```
utils/
├── __init__.py          # 包初始化文件
└── colors.py            # 颜色工具模块

tests/
└── test_colors.py       # 颜色测试脚本
```

## 技术细节

### ANSI 颜色代码

当 colorama 不可用时，使用标准 ANSI 转义序列：

| 颜色 | ANSI 代码 |
|------|-----------|
| 绿色 | `\033[92m` |
| 红色 | `\033[91m` |
| 黄色 | `\033[93m` |
| 蓝色 | `\033[94m` |
| 青色 | `\033[96m` |
| 洋红 | `\033[95m` |
| 重置 | `\033[0m` |

### 容错机制

1. **导入层面**: `try-except` 捕获 ImportError
2. **运行层面**: 提供 ANSI 代码作为后备方案
3. **显示层面**: emoji 符号在所有平台都能显示

## 常见问题

### Q: Docker 中颜色不显示？

A: 确保运行容器时使用 `-t` 参数：
```bash
docker run -it your-image
```

或在 `docker-compose.yml` 中：
```yaml
services:
  qbot:
    tty: true
```

### Q: Windows PowerShell 颜色显示异常？

A: 确保安装了 colorama：
```bash
pip install colorama
```

### Q: 如何禁用颜色？

A: 设置环境变量：
```bash
export NO_COLOR=1  # Linux/macOS
$env:NO_COLOR=1    # Windows PowerShell
```

然后修改 `colors.py` 添加检查：
```python
import os
if os.getenv('NO_COLOR'):
    # 返回纯文本
    def green(text): return text
```

## 更新日志

### v1.0.0 (2026-02-09)

- ✅ 添加跨平台彩色输出支持
- ✅ 实现智能降级机制
- ✅ 更新所有模块使用彩色输出
- ✅ 添加测试脚本
- ✅ 创建使用文档
