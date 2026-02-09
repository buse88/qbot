# 返利模块 - 京东长链接自动短链优化

## 功能概述

返利模块现已支持对 `https://item.m.jd.com` 开头的京东长链接进行自动短链接转换，然后再进行返利转链。这样可以提高转链成功率和稳定性。

## 工作流程

### 原有流程
```
用户发送京东链接 → 直接调用折京客API → 返回返利链接
```

### 优化后流程
```
用户发送 item.m.jd.com 链接 
  ↓
检测到长链接格式
  ↓
调用短链转换器 (dwz.py)
  ↓
获得短链接 (https://3.cn/xxx)
  ↓
使用短链接调用折京客API
  ↓
返回返利链接 + 口令
```

## 技术实现

### 修改文件
- **`modules/rebate/jingdong.py`**

### 关键代码

```python
# 新增：如果是 item.m.jd.com 链接，先转换为短链接
if material_url.startswith("https://item.m.jd.com"):
    if DEBUG_MODE:
        print(f"[京东转换器] 检测到 item.m.jd.com 链接，先转换为短链接")
    try:
        dwz_result = self.dwz_converter.convert(material_url, verbose=False)
        if dwz_result['success']:
            material_url = dwz_result['short_url']
            if DEBUG_MODE:
                print(f"[京东转换器] 短链转换成功: {material_url}")
        else:
            if DEBUG_MODE:
                print(f"[京东转换器] 短链转换失败，使用原链接: {dwz_result.get('error', '未知错误')}")
    except Exception as e:
        if DEBUG_MODE:
            print(f"[京东转换器] 短链转换异常，使用原链接: {str(e)}")
```

## 使用示例

### 输入
用户在群聊中发送：
```
https://item.m.jd.com/product/10144010479875.html
```

### 处理过程（DEBUG_MODE=True 时的日志）
```
[京东转换器] 检测到 item.m.jd.com 链接，先转换为短链接
[京东转换器] 短链转换成功: https://3.cn/2D-YdUAS
[京东转换器] 调用折京客API: ... with materialId=https://3.cn/2D-YdUAS
[京东转换器] 转换成功
```

### 输出
机器人回复返利信息：
```
【商品】: 商品标题

【券后】: ¥99.00

【佣金】: ¥5.00

【领券买】: https://u.jd.com/xxxxx

【领券口令】: ￥xxxxx￥

[商品图片]
```

## 容错机制

1. **短链转换失败**：如果短链转换失败，会自动使用原链接继续转链
2. **短链转换异常**：如果短链转换过程中出现异常，会捕获并使用原链接
3. **调试日志**：所有短链转换的日志都受 `DEBUG_MODE` 控制

## 优势

1. **提高成功率**：短链接格式更稳定，API 识别率更高
2. **降低错误**：避免长链接中的特殊字符导致的转链失败
3. **无缝降级**：短链转换失败时自动使用原链接，不影响用户体验
4. **性能优化**：短链接更短，减少网络传输和API处理时间

## 适用范围

- ✅ `https://item.m.jd.com/product/xxx.html`
- ✅ `https://item.m.jd.com/ware/view.action?wareId=xxx`
- ❌ `https://3.cn/xxx` (已经是短链接，不需要转换)
- ❌ `https://coupon.m.jd.com/xxx` (优惠券链接，直接返回)
- ❌ 京东口令 (不需要转换)

## 配置说明

### Sign 服务器地址

默认使用 `http://192.168.8.2:3001/sign`，如需修改，编辑 `modules/rebate/jingdong.py`：

```python
# 初始化短链转换器
self.dwz_converter = JDShortUrlConverter(sign_url="http://你的服务器:端口/sign")
```

## 注意事项

1. **依赖服务**：需要 Sign 服务器正常运行
2. **网络要求**：需要能访问 Sign 服务器和京东短链 API
3. **性能影响**：增加了一次短链转换的 API 调用，但整体转链成功率提升
4. **调试模式**：建议在测试时开启 `DEBUG_MODE=True` 查看详细日志

## 相关文件

- **京东转换器**：`modules/rebate/jingdong.py`
- **短链转换器**：`modules/news_jd/dwz.py`
- **返利模块**：`modules/rebate/module.py`
- **配置文件**：`config.py`
