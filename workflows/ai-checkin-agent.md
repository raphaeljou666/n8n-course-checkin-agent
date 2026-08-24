# AI Checkin Agent Workflow

匯出自 n8n Cloud,可透過 Settings → Import from File 匯入使用。

⚠️ 匯入後請自行到 Credentials 分頁重新設定 OpenAI、Google Sheets、Gmail 三組憑證。

## Workflow JSON

```json
{
  "name": "AI Course Check-in Agent",
  "nodes": [
    {
      "parameters": {
        "public": true,
        "initialMessages": " 歡迎來到今天的課程👏請告訴我你的姓名和班級，例如：\n「我是王小明，來報 A 班」",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.4,
      "position": [
        0,
        0
      ],
      "id": "00dc8054-cb65-4c69-bff0-78399a122162",
      "name": "When chat message received",
      "webhookId": "YOUR_WEBHOOK_ID"
    },
    {
      "parameters": {
        "options": {
          "systemMessage": "你是課程報到助手。你的任務是協助學員完成報到。\n\n流程規則:\n1. 學員會提供姓名和班級,你要先用「查詢報名資料」工具搜尋。\n2. 姓名要做模糊比對(容錯錯字、簡繁差異、空格),如果只有一筆高相似度結果就當作找到。\n3. 找到資料後,請學員確認 email 前綴或手機末四碼其中一項,做二次驗證。\n4. 驗證通過後,呼叫「更新報到狀態」工具寫入 Google Sheet。\n5. 完成報到後回覆歡迎訊息並告知教室位置(A班 201室、B班 202室、C班 203室)。\n6. 如果找不到資料或驗證三次失敗,呼叫「通知工作人員」工具寄信,並請學員稍候。\n\n\n語氣要親切、簡短,回覆使用繁體中文。"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        416,
        0
      ],
      "id": "d6719292-fcf9-4721-967b-d620a2a8d78c",
      "name": "AI Agent"
    },
    {
      "parameters": {
        "model": {
          "__rl": true,
          "value": "gpt-4o-mini",
          "mode": "list",
          "cachedResultName": "gpt-4o-mini"
        },
        "builtInTools": {},
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.3,
      "position": [
        232,
        232
      ],
      "id": "4928c804-260e-429b-b452-8fec23c6e7c0",
      "name": "OpenAI Chat Model",
      "credentials": {
        "openAiApi": {
          "id": "T8hvWfkRdmRB82WT",
          "name": "OpenAI account"
        }
      }
    },
    {
      "parameters": {},
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.4,
      "position": [
        360,
        232
      ],
      "id": "e29cf0c8-f4f9-467c-abbd-14e4701b5511",
      "name": "Simple Memory"
    },
    {
      "parameters": {
        "descriptionType": "manual",
        "toolDescription": "根據姓名或班級查詢學員報名資料,回傳報名編號、姓名、班級、Email、手機末四碼",
        "documentId": {
          "__rl": true,
          "value": "YOUR_GOOGLE_SHEET_ID",
          "mode": "list",
          "cachedResultName": "n8n_sideproject",
          "cachedResultUrl": "https://docs.google.com/spreadsheets/d/YOUR_GOOGLE_SHEET_ID/edit?usp=drivesdk"
        },
        "sheetName": {
          "__rl": true,
          "value": "gid=0",
          "mode": "list",
          "cachedResultName": "course_registration",
          "cachedResultUrl": "https://docs.google.com/spreadsheets/d/YOUR_GOOGLE_SHEET_ID/edit#gid=0"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.googleSheetsTool",
      "typeVersion": 4.7,
      "position": [
        488,
        232
      ],
      "id": "33fdd4c2-7492-43fb-a910-f5fe5caf1f2d",
      "name": "Get row(s) in sheet in Google Sheets",
      "credentials": {
        "googleSheetsOAuth2Api": {
          "id": "fCpZCzyg2DocMlEl",
          "name": "Google Sheets account"
        }
      }
    },
    {
      "parameters": {
        "descriptionType": "manual",
        "toolDescription": "將指定報名編號的報到狀態更新為「已報到」並填入報到時間",
        "operation": "update",
        "documentId": {
          "__rl": true,
          "value": "YOUR_GOOGLE_SHEET_ID",
          "mode": "list",
          "cachedResultName": "n8n_sideproject",
          "cachedResultUrl": "https://docs.google.com/spreadsheets/d/YOUR_GOOGLE_SHEET_ID/edit?usp=drivesdk"
        },
        "sheetName": {
          "__rl": true,
          "value": "gid=0",
          "mode": "list",
          "cachedResultName": "course_registration",
          "cachedResultUrl": "https://docs.google.com/spreadsheets/d/YOUR_GOOGLE_SHEET_ID/edit#gid=0"
        },
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "報名編號": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('____', ``, 'string') }}",
            "報到狀態": "已報到",
            "報到時間": "={{ $now }}"
          },
          "matchingColumns": [
            "報名編號"
          ],
          "schema": [
            {
              "id": "報名編號",
              "displayName": "報名編號",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "string",
              "canBeUsedToMatch": true,
              "removed": false
            },
            {
              "id": "姓名",
              "displayName": "姓名",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "string",
              "canBeUsedToMatch": true
            },
            {
              "id": "手機末四碼",
              "displayName": "手機末四碼",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "string",
              "canBeUsedToMatch": true
            },
            {
              "id": "Email",
              "displayName": "Email",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "string",
              "canBeUsedToMatch": true
            },
            {
              "id": "班級",
              "displayName": "班級",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "string",
              "canBeUsedToMatch": true
            },
            {
              "id": "報到狀態",
              "displayName": "報到狀態",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "string",
              "canBeUsedToMatch": true
            },
            {
              "id": "報到時間",
              "displayName": "報到時間",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "string",
              "canBeUsedToMatch": true
            },
            {
              "id": "row_number",
              "displayName": "row_number",
              "required": false,
              "defaultMatch": false,
              "display": true,
              "type": "number",
              "canBeUsedToMatch": true,
              "readOnly": true,
              "removed": true
            }
          ],
          "attemptToConvertTypes": false,
          "convertFieldsToString": false
        },
        "options": {}
      },
      "type": "n8n-nodes-base.googleSheetsTool",
      "typeVersion": 4.7,
      "position": [
        624,
        240
      ],
      "id": "4b8ba75c-4796-44f4-85ca-108ff5fbbdc1",
      "name": "Update row in sheet in Google Sheets",
      "credentials": {
        "googleSheetsOAuth2Api": {
          "id": "fCpZCzyg2DocMlEl",
          "name": "Google Sheets account"
        }
      }
    },
    {
      "parameters": {
        "descriptionType": "manual",
        "toolDescription": "當學員資料查無或驗證三次失敗時,寄信通知工作人員處理",
        "sendTo": "raphaeljou666@gmail.com",
        "subject": "[報到異常] 需要人工協助",
        "emailType": "text",
        "message": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Message', ``, 'string') }}",
        "options": {}
      },
      "type": "n8n-nodes-base.gmailTool",
      "typeVersion": 2.2,
      "position": [
        744,
        232
      ],
      "id": "9fc34902-7d51-4f42-8f98-3de11122cb36",
      "name": "Send a message in Gmail",
      "webhookId": "YOUR_WEBHOOK_ID",
      "credentials": {
        "gmailOAuth2": {
          "id": "mPAYnkaZZkDVPXgS",
          "name": "Gmail account"
        }
      }
    }
  ],
  "pinData": {},
  "connections": {
    "When chat message received": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Simple Memory": {
      "ai_memory": [
        [
          {
            "node": "AI Agent",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "Get row(s) in sheet in Google Sheets": {
      "ai_tool": [
        [
          {
            "node": "AI Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "Update row in sheet in Google Sheets": {
      "ai_tool": [
        [
          {
            "node": "AI Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "Send a message in Gmail": {
      "ai_tool": [
        [
          {
            "node": "AI Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate",
    "timeSavedMode": "fixed",
    "errorWorkflow": "NM3BCre4LGv98BnM",
    "callerPolicy": "workflowsFromSameOwner",
    "availableInMCP": false
  },
  "versionId": "3bce4182-e117-400d-ac0d-08134f1e28c9",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "2c1d39a2c741d58b5b7d20473b893eaac9994204eb79b9c48e7299386331d2f0"
  },
  "nodeGroups": [],
  "id": "SSKN1uLvnsxC27ir",
  "tags": []
}```
