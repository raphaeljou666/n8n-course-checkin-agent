# LINE Chat Bridge Workflow

作為 LINE 官方帳號和主 AI Agent 之間的橋接,把 LINE 傳來的訊息轉發給主 workflow,再把 AI 回覆送回 LINE。

## 節點結構

Webhook (POST /line-webhook)
  ↓
Code in JavaScript (解析 LINE payload,取出 userId、userMessage、replyToken)
  ↓
IF (過濾非文字訊息)
  ↓ true
Execute Sub-workflow (呼叫 AI Course Check-in Agent)
  ↓
HTTP Request (POST 到 LINE reply API 回覆訊息)

## Workflow JSON

```json
[請貼上 line-chat-bridge.json 內容,Bearer Token 請換成佔位符]
```
