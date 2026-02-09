---
description: QBot模块开发接口规范和最佳实践
---

# QBot模块开发接口规范

本文档定义了QBot项目中所有功能模块必须遵循的接口规范、最佳实践和开发指南。

## 目录

1. [模块基础结构](#模块基础结构)
2. [必须实现的接口](#必须实现的接口)
3. [生命周期钩子](#生命周期钩子)
4. [数据类规范](#数据类规范)
5. [配置管理](#配置管理)
6. [最佳实践](#最佳实践)
7. [示例模块](#示例模块)

---

## 模块基础结构

### 目录结构

```
modules/
└── your_module/
    ├── __init__.py          # 空文件或模块导出
    ├── module.py            # 主模块类(必须)
    └── [其他辅助文件]        # 可选
```

### 主模块类

所有模块必须继承`core.base_module.BaseModule`并实现必要的接口。

```python
from core.base_module import BaseModule, ModuleContext, ModuleResponse

class YourModule(BaseModule):
    """模块描述"""
    
    # 实现必要的属性和方法
    pass
```

---

## 必须实现的接口

### 1. 属性 (Properties)

#### `name` (必须)
- **类型**: `str`
- **说明**: 模块的唯一标识名称
- **示例**: `"返利模块"`, `"京东线报收集"`

```python
@property
def name(self) -> str:
    return "您的模块名称"
```

#### `version` (必须)
- **类型**: `str`
- **说明**: 模块版本号,建议使用语义化版本
- **示例**: `"1.0.0"`, `"2.1.3"`

```python
@property
def version(self) -> str:
    return "1.0.0"
```

#### `description` (必须)
- **类型**: `str`
- **说明**: 模块功能的简短描述
- **示例**: `"自动识别并转换淘宝/京东链接为推广链接"`

```python
@property
def description(self) -> str:
    return "模块功能描述"
```

#### `author` (可选)
- **类型**: `str`
- **默认值**: `"Unknown"`
- **说明**: 模块作者信息

```python
@property
def author(self) -> str:
    return "QBot Team"
```

#### `dependencies` (可选)
- **类型**: `List[str]`
- **默认值**: `[]`
- **说明**: 依赖的其他模块名称列表

```python
@property
def dependencies(self) -> List[str]:
    return ["线报数据库", "API管理器"]
```

### 2. 方法 (Methods)

#### `can_handle()` (必须)

判断模块是否能处理当前消息。

**签名**:
```python
async def can_handle(self, message: str, context: ModuleContext) -> bool:
    """
    判断是否能处理该消息
    
    Args:
        message: 清理后的消息内容(已去除CQ码)
        context: 消息上下文
        
    Returns:
        bool: 是否能处理
    """
    pass
```

**实现要点**:
1. **过滤机器人消息**: 避免机器人间互相回复
2. **群组过滤**: 只处理配置的监听群
3. **优先级检查**: 如需要,实现优先级响应逻辑
4. **内容匹配**: 使用正则表达式或关键词匹配

**示例**:
```python
async def can_handle(self, message: str, context: ModuleContext) -> bool:
    # 1. 过滤机器人消息
    if context.user_id in self.bot_qq_list:
        return False
    
    # 2. 只处理监听群
    if context.group_id not in self.watched_groups:
        return False
    
    # 3. 检查消息内容
    return bool(self.pattern.search(message))
```

#### `handle()` (必须)

处理消息并返回响应。

**签名**:
```python
async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
    """
    处理消息
    
    Args:
        message: 清理后的消息内容
        context: 消息上下文
        
    Returns:
        ModuleResponse: 模块响应,如果不需要回复则返回None
    """
    pass
```

**实现要点**:
1. **异步处理**: 使用`async/await`处理耗时操作
2. **错误处理**: 捕获异常并记录日志
3. **返回规范**: 返回`ModuleResponse`对象或`None`

**示例**:
```python
async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
    try:
        # 处理逻辑
        result = await self.process_message(message)
        
        # 返回响应
        return ModuleResponse(
            content=result,
            auto_recall=False,
            recall_delay=30
        )
    except Exception as e:
        print(f"[{self.name}] 处理失败: {e}")
        return None
```

---

## 生命周期钩子

### `on_load()` (可选但推荐)

模块加载时调用,用于初始化配置和资源。

**签名**:
```python
async def on_load(self, config: Dict[str, Any]) -> None:
    """
    模块加载时调用
    
    Args:
        config: 模块配置字典
    """
    await super().on_load(config)
    # 自定义初始化逻辑
```

**实现要点**:
1. **调用父类方法**: `await super().on_load(config)`
2. **保存配置**: `self.config = config`
3. **提取settings**: `settings = config.get('settings', {})`
4. **初始化资源**: 编译正则、加载数据等
5. **打印日志**: 输出加载信息

**示例**:
```python
async def on_load(self, config: dict) -> None:
    await super().on_load(config)
    
    # 获取配置
    self.config = config
    settings = config.get('settings', {})
    
    # 初始化
    self.watched_groups = settings.get('watched_groups', [])
    self.bot_qq_list = get_bot_qq_list()
    self.pattern = re.compile(r'your_pattern')
    
    print(f"[{self.name}] 模块已加载 (v{self.version})")
    print(f"[{self.name}] 监听群: {self.watched_groups}")
```

### 其他生命周期钩子

```python
async def on_unload(self) -> None:
    """模块卸载时调用,清理资源"""
    pass

async def on_enable(self) -> None:
    """模块启用时调用"""
    self.enabled = True

async def on_disable(self) -> None:
    """模块禁用时调用"""
    self.enabled = False
```

---

## 数据类规范

### ModuleContext

消息上下文信息。

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

模块响应数据。

```python
@dataclass
class ModuleResponse:
    content: str                      # 回复内容
    auto_recall: bool = False         # 是否自动撤回
    recall_delay: int = 30            # 撤回延迟(秒)
    quoted_msg_id: Optional[int] = None  # 引用的消息ID
    extra: Dict[str, Any] = field(default_factory=dict)  # 额外数据
```

---

## 配置管理

### 配置文件结构

在`config.py`中定义模块配置:

```python
YOUR_MODULE_CONFIG = {
    "enabled": True,           # 是否启用
    "priority": 50,            # 优先级(数字越小越高)
    "settings": {
        # 模块特定配置
        "watched_groups": [123456],
        "api_key": "your_key",
        # ...
    }
}
```

### 配置读取

```python
async def on_load(self, config: dict) -> None:
    await super().on_load(config)
    
    # 读取配置
    settings = config.get('settings', {})
    self.watched_groups = settings.get('watched_groups', [])
    self.api_key = settings.get('api_key', '')
    self.debug = config.get('debug', False)
```

### 配置辅助函数

使用`config.py`中的辅助函数:

```python
from config import get_bot_qq_list, get_bot_priority, DEBUG_MODE

# 在on_load中
self.bot_qq_list = get_bot_qq_list()
self.bot_priority = get_bot_priority()
```

---

## 最佳实践

### 1. 机器人消息过滤

**必须**过滤机器人消息以防止循环:

```python
async def can_handle(self, message: str, context: ModuleContext) -> bool:
    # 过滤所有机器人消息
    if context.user_id in self.bot_qq_list:
        if self.debug:
            print(f"[{self.name}] 跳过机器人消息: {context.user_id}")
        return False
    # ...
```

### 2. 优先级响应

如需实现优先级响应(只让优先级最高的在线机器人响应):

```python
def should_respond_by_priority(self, context: ModuleContext) -> bool:
    """判断当前机器人是否应该响应"""
    import main
    
    current_bot = context.self_id
    online_bots = main.get_online_bots()
    
    # 找出在线的优先级机器人
    online_priority_bots = [bot for bot in self.bot_priority if bot in online_bots]
    
    if not online_priority_bots:
        return True
    
    # 只有优先级最高的在线机器人响应
    return current_bot == online_priority_bots[0]
```

### 3. 调试日志

使用统一的日志格式:

```python
# 基本日志
print(f"[{self.name}] 消息内容")

# 调试日志(仅在debug模式)
if self.debug or DEBUG_MODE:
    print(f"[{self.name}] 调试信息: {data}")

# 错误日志
print(f"[{self.name}] ❌ 错误: {error}")

# 成功日志
print(f"[{self.name}] ✅ 成功: {result}")
```

### 4. 异常处理

```python
async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
    try:
        result = await self.process(message)
        return ModuleResponse(content=result)
    except Exception as e:
        print(f"[{self.name}] 处理失败: {e}")
        if DEBUG_MODE:
            import traceback
            traceback.print_exc()
        return None
```

### 5. 私聊与群聊区分

```python
async def can_handle(self, message: str, context: ModuleContext) -> bool:
    # 处理私聊消息
    if context.group_id is None:
        return self.check_private_message(message)
    
    # 处理群聊消息
    if context.group_id in self.watched_groups:
        return self.check_group_message(message)
    
    return False
```

### 6. 正则表达式编译

在`on_load`中编译正则,避免重复编译:

```python
async def on_load(self, config: dict) -> None:
    await super().on_load(config)
    
    # 编译正则表达式
    self.url_pattern = re.compile(r'https?://[^\s]+')
    self.keyword_pattern = re.compile(r'关键词1|关键词2', re.IGNORECASE)
```

---

## 示例模块

### 最小化示例

```python
"""
示例模块 - 回显消息
"""

import re
from typing import Optional
from core.base_module import BaseModule, ModuleContext, ModuleResponse
from config import get_bot_qq_list

class EchoModule(BaseModule):
    """回显模块 - 将用户消息原样返回"""
    
    @property
    def name(self) -> str:
        return "回显模块"
    
    @property
    def version(self) -> str:
        return "1.0.0"
    
    @property
    def description(self) -> str:
        return "将用户消息原样返回"
    
    @property
    def author(self) -> str:
        return "QBot Team"
    
    async def on_load(self, config: dict) -> None:
        await super().on_load(config)
        
        self.config = config
        settings = config.get('settings', {})
        
        # 配置
        self.watched_groups = settings.get('watched_groups', [])
        self.bot_qq_list = get_bot_qq_list()
        self.trigger_word = settings.get('trigger_word', '回显')
        
        print(f"[{self.name}] 模块已加载 (v{self.version})")
    
    async def can_handle(self, message: str, context: ModuleContext) -> bool:
        # 过滤机器人消息
        if context.user_id in self.bot_qq_list:
            return False
        
        # 只处理监听群
        if context.group_id not in self.watched_groups:
            return False
        
        # 检查触发词
        return message.startswith(self.trigger_word)
    
    async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
        try:
            # 移除触发词
            content = message.replace(self.trigger_word, '', 1).strip()
            
            if not content:
                return ModuleResponse(content="请在触发词后输入内容")
            
            # 返回回显
            return ModuleResponse(
                content=f"回显: {content}",
                auto_recall=False
            )
        except Exception as e:
            print(f"[{self.name}] 处理失败: {e}")
            return None
```

### 配置示例

```python
# config.py

ECHO_MODULE_CONFIG = {
    "enabled": True,
    "priority": 50,
    "settings": {
        "watched_groups": [123456, 789012],
        "trigger_word": "回显",
    }
}
```

---

## 检查清单

在提交模块前,请确认:

- [ ] 继承了`BaseModule`
- [ ] 实现了`name`, `version`, `description`属性
- [ ] 实现了`can_handle()`和`handle()`方法
- [ ] 在`can_handle`中过滤了机器人消息
- [ ] 实现了`on_load()`并正确初始化配置
- [ ] 使用了统一的日志格式`[{self.name}]`
- [ ] 添加了适当的异常处理
- [ ] 在`config.py`中添加了模块配置
- [ ] 在`main.py`的`lifespan`中加载了模块
- [ ] 测试了基本功能和边界情况

---

## 参考资料

- **BaseModule源码**: `core/base_module.py`
- **现有模块示例**:
  - 返利模块: `modules/rebate/module.py`
  - 京东线报收集: `modules/news_jd/module.py`
  - 线报转发: `modules/news_forwarder/module.py`
