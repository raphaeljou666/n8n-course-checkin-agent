# LINE 整合實作筆記

實作 LINE Messaging API 整合時遇到的問題和解法。

## 問題 1:主 workflow 搜不到

Execute Sub-workflow 節點裡搜尋不到主 workflow。

**原因**:主 workflow 沒 Publish/Active,別的 workflow 就查不到它。

**解法**:主 workflow 右上角 Publish 按下去,變綠色再回來搜。

## 問題 2:AI Agent 出現 "No prompt specified"

LINE 訊息傳進來後 AI Agent 抓不到訊息內容。

**原因**:LINE Bridge 傳過來的欄位名叫 `userMessage`,不是預設的 `chatInput`。

**解法**:
- AI Agent 的 Prompt 改成 `{{ $json.userMessage }}`
- Simple Memory 的 Session ID 改成 `{{ $json.userId }}`
- 或者主 workflow 用兩個 Trigger 並存(Chat Trigger + Execute Workflow Trigger),配合 Accept all data 模式

## 問題 3:HTTP Request 回覆 LINE 時 JSON parse 錯誤

錯誤訊息:`Bad control character in string literal in JSON at position XXX`

**原因**:AI 回覆內容裡有 `\n` 換行符,直接塞進 JSON 字串會被 parse 器拒絕。

**解法**:在 JSON Body 裡把換行符替換掉:

```json
"text": "{{ $json.output.replaceAll('\n', ' ') }}"
```

## 問題 4:Verify Webhook 一直失敗

原因通常有兩個:

1. LINE Chat Bridge workflow 沒 Publish
2. Webhook URL 用到 Test URL(網址含 `/webhook-test/`),要改用 Production URL(網址是 `/webhook/`)

## 敏感資訊警告

實作過程中若截圖或分享 LINE Developers Console 內容,務必遮住:

- Channel Secret
- Channel Access Token

一旦外洩必須立刻到 LINE Developers Console 點 Issue 重新產生新的一組。
