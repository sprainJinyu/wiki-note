---
title: Dify 对话流与前端 SSE 对接协议
created: 2026-09-14
updated: 2026-09-14
tags: [Dify, SSE, 流式协议, 前端对接, ChatX, 状态机]
---

# 💬 Dify 对话流与前端 SSE 对接协议

> 本文档详细拆解从轻舟 CRM 触发 Dify 智能体对话，到 `member-ai-center` 中转组装，再到前端流式逐字渲染的全过程与协议报文格式。

---

## 一、 前端接入与宿主通信机制

海豚 AI 在轻舟 CRM 等后台系统中并非直接内嵌 React 组件，而是采用 **独立微前端 SDK + iframe 沙箱隔离** 的设计。

### 1. 宿主加载与初始化（`crm-backend-ui`）
在 [`crm-backend-ui/src/main.js#L274-L293`](file:///Users/zjy/Developer/work/topsports/crm-backend-ui/src/main.js#L274-L293) 中，用户登录 CRM 系统后触发初始化：

```javascript
function initDolphinAI() {
  const matched = location.host.match(/(-(test|dev))\./)
  window.embedChatbot && window.embedChatbot({
    // 动态路由至测试环境或生产环境
    baseUrl: `https://dolphin-ai${matched ? matched[1] : ""}.topsports.com.cn`,
    async login() {
      // 通过 CRM 接口获取当前登录人的海豚 AI token
      const res = await request({
        method: "post",
        url: "/dolphin/ai/auth/getToken"
      })
      window.dolphinAILoginData = res.data
      return res.data
    },
    track: window.dolphinAiTrack // 挂载埋点上报
  })
}
```

### 2. SDK 核心交互协议（`pc-dolphin-ai/sdk/index.js`）
SDK 在页面右下角渲染悬浮气泡按钮 `#chatbot-bubble-button`。用户点击后拉起 iframe，宿主与 iframe 之间通过标准的 `window.postMessage` 建立通信：
* **`loaded`**：iframe 加载完成，宿主回送 `init` 事件携带 `token` 与 `clientId`；
* **`ask`**：宿主向智能体直接灌入预设问题并拉起会话窗口；
* **`close`**：收起右侧浮动抽屉；
* **`refresh` / `auth-expired`**：登录态过期，通知宿主重新调用 `login()` 刷新 token。

---

## 二、 前端请求报文规范

在 `pc-dolphin-ai/components/Chat/ChatX.tsx` 中，用户提交问题触发网络请求：

* **接口路径**：`POST /aigc-api/chat/see/chat-messages`（通过网关映射到后端的 `/frontend/ai/chat/see/chat-messages`）
* **Content-Type**：`application/json;charset=UTF-8`
* **Accept / Produces**：`text/event-stream`
* **自定义请求头**：
  * `Authorization`: `Bearer {token}`
  * `x-request-env-flag`: 灰度环境标识（用于后端灰度流量染色）

### 请求体示例（JSON）
```json
{
  "question": "帮我查询一下上个月华东区的销售业绩汇总",
  "responseMode": "streaming",
  "agentId": "bot-66f123456789abcdef",
  "useChannel": 1,
  "clientId": "",
  "conversationId": "conv-a1b2c3d4-5678-90ef",
  "files": [
    {
      "id": "file-101",
      "fileName": "销售报表.xlsx",
      "fileUrl": "https://member-oss-cdn.topsports.com.cn/chat/file-101.xlsx"
    }
  ],
  "customParamsMap": {
    "storeCode": "TS001",
    "region": "EastChina"
  }
}
```

---

## 三、 后端中枢流式调度（`member-ai-center`）

### 1. 控制器入口
控制器代码位于 [`ChatController.java#L66-L76`](file:///Users/zjy/Developer/work/topsports/member-ai-center/member-ai-controller/src/main/java/com/topsports/member/ai/controller/frontend/chat/ChatController.java#L66-L76)：
```java
@PostMapping(path = "/see/chat-messages", produces = {MediaType.TEXT_EVENT_STREAM_VALUE})
public Flux<String> seeChatMessage(@RequestBody ChatRequestCmd cmd, HttpServletRequest request) {
    String envFlag = request.getHeader("x-request-env-flag");
    GrayFlagContext.set(envFlag);
    try {
        return chatFacadeService.chatMessageStream(cmd);
    } finally {
        GrayFlagContext.clear();
    }
}
```

### 2. DifyChatService 执行逻辑
1. **策略分发**：根据 `agent.getLlmAppSource()` 匹配到 `DifyChatService`。
2. **会话参数富化（`sessionParamAssignCalc`）**：自动拉取并计算当前用户的员工工号、门店元数据，合并入 `customParamsMap`。
3. **灰度流量染色**：如果命中配置的灰度环境，后端自动在下游 Dify URL 追加 `?grayTraceId={traceId}`，确保链路上下一体可溯。
4. **调用 Dify 流式接口**：通过 OkHttp/HttpClient 建立对 Dify 的 SSE 连接，接收 Dify 产出的流式数据块。

---

## 四、 Dify SSE 响应报文规范

后端将 Dify 的响应转换为标准 SSE 格式推回给前端，每帧格式为 `data: {JSON}\n\n`。

### 1. 常见事件类型
* **`message`**：常规文本增量流；
* **`message_replace`**：覆盖替换（工作流某些特殊节点或内容审核安全兜底，会要求前端全量覆盖前文，而非追加拼接）；
* **`chat_notice`**：系统提示（如超过今日使用额度、会话过期、降级提示，前端渲染为居中提示条或分割线）；
* **`[DONE]`**：流式传输完毕标识。

### 2. 报文示例

#### 片段 1：常规增量回答（`message`）
```text
data: {"event":"message","taskId":"task-001","messageId":"msg-999","conversationId":"conv-a1b2c3d4-5678-90ef","answer":"上个月","createdAt":1726300000}

data: {"event":"message","taskId":"task-001","messageId":"msg-999","conversationId":"conv-a1b2c3d4-5678-90ef","answer":"华东区销售额达","createdAt":1726300001}

```

#### 片段 2：整句覆盖替换（`message_replace`）
```text
data: {"event":"message_replace","taskId":"task-001","messageId":"msg-999","conversationId":"conv-a1b2c3d4-5678-90ef","answer":"经复核修正：上个月华东区销售总额为 1,230 万元。","createdAt":1726300005}

```

#### 片段 3：通知与结束（`chat_notice` 与 `[DONE]`）
```text
data: {"event":"chat_notice","answer":"今日可用额度还剩 5 次","createdAt":1726300006}

data: [DONE]

```

---

## 五、 前端流式消费状态机（`ChatX.tsx`）

前端的核心逻辑集中在 [`ChatX.tsx#L750-L793`](file:///Users/zjy/Developer/work/topsports/pc-dolphin-ai/components/Chat/ChatX.tsx#L750-L793) 与 [`ChatX.tsx#L699-L748`](file:///Users/zjy/Developer/work/topsports/pc-dolphin-ai/components/Chat/ChatX.tsx#L699-L748)。

### 1. 流读取与粘包拆包（`consumeSseResponse`）
浏览器 `ReadableStream` 传输时，一个 chunk 可能包含半条消息或多条消息。前端通过自维护的 `buffer` 及正则 `\r?\n\r?\n` 解决粘包切分：
```typescript
async function consumeSseResponse(response: Response, onData: (data: string) => void) {
  const reader = response.body!.getReader()
  const decoder = new TextDecoder()
  let buffer = ""

  const flushEvent = (rawEvent: string) => {
    // 过滤出所有以 data: 开头的有效载荷
    const data = rawEvent
      .split(/\r?\n/)
      .filter((line) => line.startsWith("data:"))
      .map((line) => line.slice(5).trimStart())
      .join("\n")
    if (data) onData(data)
  }

  while (true) {
    const { done, value } = await reader.read()
    buffer += decoder.decode(value, { stream: !done })

    let separatorIndex = buffer.search(/\r?\n\r?\n/)
    while (separatorIndex >= 0) {
      const rawEvent = buffer.slice(0, separatorIndex)
      const separatorLength = buffer[separatorIndex] === "\r" ? 4 : 2
      buffer = buffer.slice(separatorIndex + separatorLength)
      flushEvent(rawEvent)
      separatorIndex = buffer.search(/\r?\n\r?\n/)
    }
    if (done) break
  }
}
```

### 2. 状态合并与消息构建（`mergeStreamMessage`）
```typescript
function mergeStreamMessage(originMessage: ChatMessage | undefined, chunkJson: StreamPayload): ChatMessage {
  const content = typeof originMessage?.content === "string" ? originMessage.content : ""

  // 1. 全量替换事件
  if (chunkJson.event === "message_replace") {
    return {
      ...originMessage,
      content: chunkJson.answer || "",
      event: "message_replace",
      role: "assistant"
    }
  }

  // 2. 系统通知分割线事件
  if (chunkJson.event === "chat_notice") {
    return {
      content: chunkJson.answer || "",
      role: "divider",
      event: "chat_notice"
    }
  }

  // 3. 正常增量拼接
  return {
    ...originMessage,
    content: `${content}${chunkJson.answer || ""}`,
    role: "assistant"
  }
}
```

通过这一套状态机机制，实现了对大模型工作流输出、复杂重写、多行增量流的平滑渲染。