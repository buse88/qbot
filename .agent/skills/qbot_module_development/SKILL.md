---
name: QBot模块开发规范
description: QBot项目模块开发的接口规范、最佳实践和开发指南。所有新模块必须遵循此规范。
---

# QBot模块开发规范

## 概述

本skill定义了QBot项目中所有功能模块必须遵循的接口规范和最佳实践。在开发新模块前,**必须**完整阅读本文档。

## 核心原则

1. **统一接口**: 所有模块继承`BaseModule`并实现标准接口
2. **防止循环**: 必须过滤机器人消息,避免机器人间互相回复
3. **优先级响应**: 支持多机器人优先级机制
4. **异常处理**: 完善的错误处理和日志记录
5. **配置驱动**: 所有可变参数通过配置文件管理

---

## 模块结构

### 目录结构

```
modules/
└── your_module/
    ├── __init__.py          # 空文件
    ├── module.py            # 主模块类(必须)
    └── [其他文件]            # 可选辅助文件
```

### 主模块类模板

```python
"""
模块名称 - 简短描述
"""

import re
from typing import Optional
from core.base_module import BaseModule, ModuleContext, ModuleResponse
from config import get_bot_qq_list, DEBUG_MODE

class YourModule(BaseModule):
    """模块功能详细描述"""
    
    # ===== 必须实现的属性 =====
    
    @property
    def name(self) -> str:
        return "模块名称"
    
    @property
    def version(self) -> str:
        return "1.0.0"
    
    @property
    def description(self) -> str:
        return "模块功能描述"
    
    @property
    def author(self) -> str:
        return "QBot Team"
    
    # ===== 生命周期钩子 =====
    
    async def on_load(self, config: dict) -> None:
        """模块加载时初始化"""
        await super().on_load(config)
        
        # 保存配置
        self.config = config
        settings = config.get('settings', {})
        
        # 获取机器人列表(必须)
        self.bot_qq_list = get_bot_qq_list()
        
        # 初始化配置
        self.watched_groups = settings.get('watched_groups', [])
        
        # 编译正则表达式
        self.pattern = re.compile(r'your_pattern')
        
        print(f"[{self.name}] 模块已加载 (v{self.version})")
    
    # ===== 必须实现的方法 =====
    
    async def can_handle(self, message: str, context: ModuleContext) -> bool:
        """判断是否能处理该消息"""
        
        # 1. 过滤机器人消息(必须)
        if context.user_id in self.bot_qq_list:
            if DEBUG_MODE:
                print(f"[{self.name}] 跳过机器人消息: {context.user_id}")
            return False
        
        # 2. 私聊处理(可选)
        if context.group_id is None:
            return self.check_private_message(message)
        
        # 3. 群组过滤
        if context.group_id not in self.watched_groups:
            return False
        
        # 4. 内容匹配
        return bool(self.pattern.search(message))
    
    async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
        """处理消息并返回响应"""
        try:
            # 处理逻辑
            result = await self.process_message(message, context)
            
            # 返回响应
            return ModuleResponse(
                content=result,
                auto_recall=False,
                recall_delay=30
            )
        except Exception as e:
            print(f"[{self.name}] ❌ 处理失败: {e}")
            if DEBUG_MODE:
                import traceback
                traceback.print_exc()
            return None
```

---

## 必须实现的接口

### 属性 (Properties)

| 属性 | 类型 | 必须 | 说明 | 示例 |
|------|------|------|------|------|
| `name` | `str` | ✅ | 模块唯一标识 | `"返利模块"` |
| `version` | `str` | ✅ | 语义化版本号 | `"1.0.0"` |
| `description` | `str` | ✅ | 功能描述 | `"自动转换推广链接"` |
| `author` | `str` | 📝 | 作者信息 | `"QBot Team"` |
| `dependencies` | `List[str]` | 📝 | 依赖模块列表 | `["数据库模块"]` |

### 方法 (Methods)

#### `can_handle(message, context) -> bool`

**必须实现**。判断模块是否能处理当前消息。

**关键检查项**:
1. ✅ **过滤机器人消息**: `if context.user_id in self.bot_qq_list: return False`
2. ✅ **群组过滤**: 只处理配置的监听群
3. ✅ **内容匹配**: 使用正则或关键词匹配

#### `handle(message, context) -> Optional[ModuleResponse]`

**必须实现**。处理消息并返回响应。

**关键要点**:
1. ✅ **异步处理**: 使用`async/await`
2. ✅ **异常捕获**: `try-except`包裹所有逻辑
3. ✅ **日志记录**: 使用统一格式`[{self.name}]`

---

## 最佳实践(必须遵循)

### 1. 机器人消息过滤(强制)

**必须**在`can_handle`开头过滤机器人消息:

```python
async def can_handle(self, message: str, context: ModuleContext) -> bool:
    # 过滤所有机器人消息(防止循环)
    if context.user_id in self.bot_qq_list:
        if DEBUG_MODE:
            print(f"[{self.name}] 跳过机器人消息: {context.user_id}")
        return False
    # ... 其他逻辑
```

### 2. 优先级响应(推荐)

如需实现多机器人优先级响应:

```python
from config import get_bot_priority
import main

def should_respond_by_priority(self, context: ModuleContext) -> bool:
    """只让优先级最高的在线机器人响应"""
    current_bot = context.self_id
    online_bots = main.get_online_bots()
    
    # 找出在线的优先级机器人
    online_priority_bots = [bot for bot in self.bot_priority if bot in online_bots]
    
    if not online_priority_bots:
        return True
    
    # 只有优先级最高的在线机器人响应
    return current_bot == online_priority_bots[0]

async def can_handle(self, message: str, context: ModuleContext) -> bool:
    # ... 其他检查
    
    # 优先级检查
    if not self.should_respond_by_priority(context):
        return False
    
    # ...
```

### 3. 统一日志格式(强制)

```python
# 基本日志
print(f"[{self.name}] 消息内容")

# 调试日志
if DEBUG_MODE:
    print(f"[{self.name}] 调试: {data}")

# 错误日志
print(f"[{self.name}] ❌ 错误: {error}")

# 成功日志
print(f"[{self.name}] ✅ 成功: {result}")
```

### 4. 异常处理(强制)

```python
async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
    try:
        result = await self.process(message)
        return ModuleResponse(content=result)
    except Exception as e:
        print(f"[{self.name}] ❌ 处理失败: {e}")
        if DEBUG_MODE:
            import traceback
            traceback.print_exc()
        return None
```

### 5. 正则编译优化(推荐)

在`on_load`中编译正则,避免重复编译:

```python
async def on_load(self, config: dict) -> None:
    await super().on_load(config)
    
    # 编译正则表达式
    self.url_pattern = re.compile(r'https?://[^\s]+')
    self.keyword_pattern = re.compile(r'关键词1|关键词2', re.IGNORECASE)
```

---

## 配置管理

### config.py配置结构

```python
YOUR_MODULE_CONFIG = {
    "enabled": True,           # 是否启用
    "priority": 50,            # 优先级(越小越高)
    "settings": {
        # 模块特定配置
        "watched_groups": [123456],
        "bot_priority": [435438881, 3121201314],  # 如需优先级响应
        "api_key": "your_key",
        # ...
    }
}
```

### 配置读取

```python
async def on_load(self, config: dict) -> None:
    await super().on_load(config)
    
    settings = config.get('settings', {})
    self.watched_groups = settings.get('watched_groups', [])
    self.api_key = settings.get('api_key', '')
```

### 使用配置辅助函数

```python
from config import get_bot_qq_list, get_bot_priority, DEBUG_MODE

# 在on_load中
self.bot_qq_list = get_bot_qq_list()
self.bot_priority = get_bot_priority()  # 如需优先级响应
```

---

## 数据类规范

### ModuleContext

```python
@dataclass
class ModuleContext:
    group_id: Optional[int]      # 群号,私聊为None
    user_id: int                 # 发送者QQ号
    message_id: Optional[int]    # 消息ID
    self_id: int                 # 当前机器人QQ号
    ws: Any                      # WebSocket对象
    raw_message: str             # 原始消息(含CQ码)
    extra: Dict[str, Any]        # 额外数据
```

### ModuleResponse

```python
@dataclass
class ModuleResponse:
    content: str                      # 回复内容
    auto_recall: bool = False         # 是否自动撤回
    recall_delay: int = 30            # 撤回延迟(秒)
    quoted_msg_id: Optional[int] = None  # 引用的消息ID
    extra: Dict[str, Any] = field(default_factory=dict)
```

---

## 模块加载流程

### 1. 在config.py中添加配置

```python
YOUR_MODULE_CONFIG = {
    "enabled": True,
    "priority": 50,
    "settings": { ... }
}
```

### 2. 在main.py的lifespan中加载

```python
from config import YOUR_MODULE_CONFIG

# 在lifespan函数中
if YOUR_MODULE_CONFIG.get('enabled', True):
    try:
        await module_loader.load_module_from_path('modules/your_module', YOUR_MODULE_CONFIG)
        print("[系统] 您的模块加载成功")
    except Exception as e:
        print(f"[系统] 您的模块加载失败: {e}")
```

---

## 开发检查清单

提交模块前,请确认:

- [ ] 继承了`BaseModule`
- [ ] 实现了`name`, `version`, `description`属性
- [ ] 实现了`can_handle()`和`handle()`方法
- [ ] **在`can_handle`开头过滤了机器人消息**
- [ ] 实现了`on_load()`并正确初始化配置
- [ ] 使用了统一的日志格式`[{self.name}]`
- [ ] 添加了`try-except`异常处理
- [ ] 在`config.py`中添加了模块配置
- [ ] 在`main.py`的`lifespan`中加载了模块
- [ ] 测试了基本功能和边界情况
- [ ] 测试了机器人消息过滤(发送机器人消息,确认不会回复)

---

## 参考示例

### 现有模块

- **BaseModule**: `core/base_module.py` - 基类定义
- **返利模块**: `modules/rebate/module.py` - 完整实现,包含优先级响应
- **京东线报**: `modules/news_jd/module.py` - 数据库交互,API调用
- **线报转发**: `modules/news_forwarder/module.py` - 后台任务,多账号轮询

### 最小化示例

参考文档中的`EchoModule`示例,包含所有必需接口的最简实现。

---

## 常见错误

### ❌ 忘记过滤机器人消息

```python
# 错误示例
async def can_handle(self, message: str, context: ModuleContext) -> bool:
    return bool(self.pattern.search(message))  # 会导致机器人间循环!
```

```python
# 正确示例
async def can_handle(self, message: str, context: ModuleContext) -> bool:
    if context.user_id in self.bot_qq_list:  # 必须先过滤
        return False
    return bool(self.pattern.search(message))
```

### ❌ 未导入DEBUG_MODE

```python
# 错误示例
if DEBUG_MODE:  # NameError: name 'DEBUG_MODE' is not defined
    print("debug")
```

```python
# 正确示例
from config import DEBUG_MODE

if DEBUG_MODE:
    print("debug")
```

### ❌ 未处理异常

```python
# 错误示例
async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
    result = await self.api_call(message)  # 可能抛出异常
    return ModuleResponse(content=result)
```

```python
# 正确示例
async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
    try:
        result = await self.api_call(message)
        return ModuleResponse(content=result)
    except Exception as e:
        print(f"[{self.name}] ❌ 处理失败: {e}")
        return None
```

---

## 总结

开发新模块时,请严格遵循以下流程:

1. ✅ 阅读本skill文档
2. ✅ 参考现有模块示例
3. ✅ 创建模块目录和文件
4. ✅ 实现所有必需接口
5. ✅ **确保过滤机器人消息**
6. ✅ 添加配置到`config.py`
7. ✅ 在`main.py`中加载模块
8. ✅ 完成检查清单
9. ✅ 测试功能和边界情况
10. ✅ 提交代码

**记住**: 机器人消息过滤是**强制性**的,忘记过滤会导致严重的消息循环问题!
