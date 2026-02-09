# 京东短链转换功能使用说明

## 功能概述

QBot 现已支持京东长链接转短链接功能，通过 `dwz` 指令可以快速将京东商品链接转换为短链接。

## 使用方法

### 基本语法

```
dwz 京东链接
```

### 示例

**输入**：
```
dwz https://item.m.jd.com/product/10144010479875.html
```

**输出**：
```
✅ 短链接转换成功
https://3.cn/2D-YdUAS
```

## 功能特点

1. **持久显示**：机器人的回复消息不会自动撤回，方便用户复制短链接
2. **权限控制**：只有配置的管理员才能使用此功能
3. **群组过滤**：只在配置的监听群中响应
4. **优先级控制**：遵循机器人优先级配置，避免重复响应

## 技术实现

### 依赖服务

- **Sign 服务器**：`http://192.168.8.2:3001/sign`
  - 用于生成京东 API 所需的签名参数
  
### 转换流程

1. 用户发送 `dwz` 指令 + 京东链接
2. 群管理模块捕获指令
3. 调用 `JDShortUrlConverter` 进行转换
   - 请求 Sign 服务器获取签名
   - 调用京东短链 API
4. 返回短链接结果（不自动撤回）

### 错误处理

如果转换失败，会返回错误信息：

```
❌ 转换失败: Sign 接口调用失败
```

或

```
❌ 转换异常: 网络超时
```

## 配置说明

### 修改 Sign 服务器地址

如需修改 Sign 服务器地址，编辑 `modules/group_admin/module.py`：

```python
# 初始化京东短链转换器
self.jd_converter = JDShortUrlConverter(sign_url="http://你的服务器:端口/sign")
```

## 注意事项

1. **网络要求**：需要能访问 Sign 服务器和京东 API
2. **链接格式**：仅支持京东商品链接（`https://item.m.jd.com/...` 或 `https://item.jd.com/...`）
3. **权限要求**：使用者必须在 `config.py` 的 `admin_qq_list` 中配置
4. **群组限制**：只在 `watched_groups` 配置的群中生效

## 相关文件

- **转换器实现**：`modules/news_jd/dwz.py`
- **群管理模块**：`modules/group_admin/module.py`
- **配置文件**：`config.py`
- **依赖列表**：`requirements.txt`
