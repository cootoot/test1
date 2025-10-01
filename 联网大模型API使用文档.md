# 支持联网搜索的大模型API调用文档

## 概述

本文档介绍如何调用支持联网搜索功能的大模型API，主要基于阿里云通义千问模型实现。通过启用联网搜索功能，AI可以获取实时信息、最新数据和当前事件，而不仅仅依赖训练数据。

## API基本信息

### 服务商：阿里云通义千问
- **API地址**: `https://dashscope.aliyuncs.com/api/v1/services/aigc/text-generation/generation`
- **支持模型**: qwen-plus, qwen-turbo, qwen-max 等
- **认证方式**: Bearer Token
- **联网搜索**: 通过 `enable_search: true` 参数启用

## 核心配置参数

### 标准请求格式
```json
{
    "model": "qwen-plus",
    "input": {
        "messages": [
            {"role": "system", "content": "你是一个AI助手，需要时可以进行联网搜索获取最新信息"},
            {"role": "user", "content": "用户的问题"}
        ]
    },
    "parameters": {
        "result_format": "message",
        "top_p": 0.8,
        "temperature": 0.7,
        "enable_search": true
    }
}
```

### 关键参数说明
- **enable_search**: `true` - 启用联网搜索功能（核心参数）
- **model**: 推荐使用 `qwen-plus` 或 `qwen-max`
- **temperature**: 0.1-1.0，控制回答的随机性
- **top_p**: 0.1-1.0，控制回答的多样性
- **result_format**: 固定为 `"message"`

## 联网搜索能力

### 可获取的信息类型
- **实时数据**: 当前时间、日期、天气
- **最新价格**: 股票、加密货币、商品价格
- **新闻事件**: 最新新闻、热点事件
- **实时状态**: 网站状态、服务可用性
- **最新资讯**: 政策更新、技术发展

### 搜索触发条件
AI会在以下情况自动进行联网搜索：
- 用户询问"最新"、"现在"、"今天"等时间相关信息
- 查询实时价格、股价等动态数据
- 询问最近发生的事件或新闻
- 需要验证当前状态的信息

## 代码实现示例

### JavaScript实现
```javascript
async function callQwenWithSearch(userMessage) {
    const API_KEY = "sk-5cc4c33112934b80b043e8f34a402f24";
    const API_URL = "https://dashscope.aliyuncs.com/api/v1/services/aigc/text-generation/generation";
    
    const requestBody = {
        model: "qwen-plus",
        input: {
            messages: [
                {
                    role: "system", 
                    content: "你是一个AI助手，可以进行联网搜索获取最新信息"
                },
                {
                    role: "user", 
                    content: userMessage
                }
            ]
        },
        parameters: {
            result_format: "message",
            top_p: 0.8,
            temperature: 0.7,
            enable_search: true  // 启用联网搜索
        }
    };
    
    try {
        const response = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${API_KEY}`
            },
            body: JSON.stringify(requestBody)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        return data.output.choices[0].message.content;
        
    } catch (error) {
        console.error('API调用失败:', error);
        throw error;
    }
}

// 使用示例
callQwenWithSearch("今天的日期是什么？比特币现在的价格是多少？")
    .then(result => console.log(result))
    .catch(error => console.error(error));
```

### Python实现
```python
import requests
import json

def call_qwen_with_search(user_message, api_key):
    """
    调用支持联网搜索的通义千问API
    """
    url = "https://dashscope.aliyuncs.com/api/v1/services/aigc/text-generation/generation"
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}"
    }
    
    data = {
        "model": "qwen-plus",
        "input": {
            "messages": [
                {
                    "role": "system",
                    "content": "你是一个AI助手，可以进行联网搜索获取最新信息"
                },
                {
                    "role": "user",
                    "content": user_message
                }
            ]
        },
        "parameters": {
            "result_format": "message",
            "top_p": 0.8,
            "temperature": 0.7,
            "enable_search": True  # 启用联网搜索
        }
    }
    
    try:
        response = requests.post(url, headers=headers, json=data, timeout=120)
        response.raise_for_status()
        
        result = response.json()
        return result["output"]["choices"][0]["message"]["content"]
        
    except requests.exceptions.RequestException as e:
        print(f"API调用失败: {e}")
        raise

# 使用示例
api_key = "sk-5cc4c33112934b80b043e8f34a402f24"
result = call_qwen_with_search("今天北京的天气如何？", api_key)
print(result)
```

## 错误处理和重试机制

### 推荐的错误处理策略
```javascript
async function robustApiCall(userMessage, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await callQwenWithSearch(userMessage);
        } catch (error) {
            console.log(`第${i + 1}次尝试失败:`, error.message);
            
            if (i === maxRetries - 1) throw error;
            
            // 指数退避重试
            await new Promise(resolve => 
                setTimeout(resolve, Math.pow(2, i) * 1000)
            );
        }
    }
}
```

### 常见错误码
- **400**: 请求参数错误
- **401**: API密钥无效或过期
- **429**: 请求频率超限
- **500**: 服务器内部错误
- **超时**: 联网搜索耗时较长，建议设置120秒以上超时

## 最佳实践

### 1. 提示词优化
```javascript
// 好的提示词示例
const systemPrompt = `你是一个专业的AI助手。当用户询问需要最新信息的问题时，请主动进行联网搜索。
搜索时请关注权威来源，并在回答中注明信息来源和时间。`;

// 用户问题示例
const userQuestions = [
    "今天的股市表现如何？",
    "最新的AI技术发展趋势是什么？",
    "现在比特币的价格是多少？",
    "今天有什么重要新闻？"
];
```

### 2. 性能优化建议
- 设置合理的超时时间（建议120-180秒）
- 实现重试机制处理网络异常
- 对于不需要实时信息的问题，可以不启用联网搜索以提高响应速度
- 缓存频繁查询的结果以减少API调用

### 3. 成本控制
- 联网搜索会消耗更多tokens，注意控制调用频率
- 根据实际需要选择是否启用联网搜索
- 监控API使用量和费用

## 测试验证

### 联网搜索功能测试
```javascript
// 测试联网搜索是否正常工作
async function testWebSearch() {
    const testQuestions = [
        "今天是几月几号？",
        "现在北京时间是多少？",
        "比特币当前价格是多少？",
        "今天有什么重要新闻？"
    ];
    
    for (const question of testQuestions) {
        try {
            const result = await callQwenWithSearch(question);
            console.log(`问题: ${question}`);
            console.log(`回答: ${result}\n`);
        } catch (error) {
            console.error(`测试失败: ${question}`, error);
        }
    }
}
```

## 注意事项

### API使用限制
- 需要有效的阿里云通义千问API密钥
- API密钥需要开启联网搜索权限
- 注意API调用频率限制和配额管理
- 联网搜索功能可能产生额外费用

### 网络和性能
- 联网搜索响应时间较长（通常30-120秒）
- 需要稳定的网络连接
- 建议在服务端调用以避免跨域问题
- 对于实时性要求高的应用，考虑使用WebSocket或Server-Sent Events

### 安全考虑
- 不要在客户端代码中暴露API密钥
- 建议通过后端代理进行API调用
- 对用户输入进行适当的验证和过滤

---

*本文档提供了调用支持联网搜索的大模型API的完整指南，适用于各种开发场景。记得根据实际需求调整参数和错误处理策略。*
